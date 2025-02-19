import SwiftUI

struct SmartPassPinView: View {
    @State private var pin: String = ""
    @State private var confirmPin: String = ""
    @State private var showPinRestrictions = false
    @State private var isPinValid = false
    @State private var errorMessage: String?

    var body: some View {
        VStack {
            Text("Set up your Smart Pass PIN")
                .font(.title2)
                .fontWeight(.bold)
                .padding(.top, 40)
            
            Text("Choose a new 4-digit PIN to make transactions, submit requests, and access security settings")
                .font(.body)
                .multilineTextAlignment(.center)
                .padding(.horizontal, 20)
            
            Spacer()

            // PIN Entry Fields
            VStack(spacing: 20) {
                SecureField("Enter PIN", text: $pin)
                    .keyboardType(.numberPad)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .multilineTextAlignment(.center)
                    .onChange(of: pin) { _ in
                        validatePin()
                    }
                
                SecureField("Confirm PIN", text: $confirmPin)
                    .keyboardType(.numberPad)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .multilineTextAlignment(.center)
                    .onChange(of: confirmPin) { _ in
                        validatePin()
                    }
            }
            .padding(.horizontal, 40)

            // Error Message
            if let errorMessage = errorMessage {
                Text(errorMessage)
                    .foregroundColor(.red)
                    .font(.footnote)
                    .padding(.top, 5)
            }

            // PIN Restrictions Link
            Button(action: {
                showPinRestrictions.toggle()
            }) {
                Text("View Smart Pass PIN guidelines")
                    .font(.footnote)
                    .foregroundColor(.blue)
            }
            .padding(.top, 10)
            
            // Continue Button
            Button(action: {
                // Handle PIN Submission
                print("PIN Set Successfully")
            }) {
                Text("Continue")
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(isPinValid ? Color.blue : Color.gray)
                    .foregroundColor(.white)
                    .cornerRadius(8)
            }
            .padding(.horizontal, 40)
            .padding(.top, 20)
            .disabled(!isPinValid)

            Spacer()
        }
        .sheet(isPresented: $showPinRestrictions) {
            PinRestrictionsView()
        }
    }
    
    func validatePin() {
        let rules = PinValidationRules()
        if pin == confirmPin {
            if rules.isValid(pin: pin) {
                isPinValid = true
                errorMessage = nil
            } else {
                isPinValid = false
                errorMessage = "PIN does not meet the required criteria."
            }
        } else {
            isPinValid = false
            errorMessage = "PIN and confirmation PIN do not match."
        }
    }
}
import Foundation

struct PinValidator {
    // Regular expressions to match forbidden patterns
    private let forbiddenPatterns: [NSRegularExpression] = [
        try! NSRegularExpression(pattern: "^(\\d)\\1{3}$"), // Repeating numbers (e.g., 1111)
        try! NSRegularExpression(pattern: "^(0123|1234|2345|3456|4567|5678|6789|7890)$"), // Sequential numbers
        try! NSRegularExpression(pattern: "^(9876|8765|7654|6543|5432|4321|3210)$"), // Reverse sequential numbers
        try! NSRegularExpression(pattern: "^(\\d)(\\d)\\2\\1$") // Mirrored numbers (e.g., 1221)
    ]
    
    // Function to validate the PIN
    func isValid(pin: String) -> Bool {
        // Ensure the PIN is exactly 4 digits
        guard pin.count == 4, pin.allSatisfy({ $0.isNumber }) else { return false }
        
        // Check against each forbidden pattern
        for pattern in forbiddenPatterns {
            let range = NSRange(location: 0, length: pin.utf16.count)
            if pattern.firstMatch(in: pin, options: [], range: range) != nil {
                return false
            }
        }
        return true
    }
}

import XCTest
@testable import YourAppModuleName

final class PinValidationTests: XCTestCase {
    var validator: PinValidationRules!

    override func setUp() {
        super.setUp()
        validator = PinValidationRules()
    }

    func testValidPin() {
        XCTAssertTrue(validator.isValid(pin: "2580"), "PIN '2580' should be valid.")
    }

    func testInvalidPinWithLetters() {
        XCTAssertFalse(validator.isValid(pin: "12a4"), "PIN '12a4' should be invalid due to letters.")
    }

    func testInvalidPinWithLessThanFourDigits() {
        XCTAssertFalse(validator.isValid(pin: "123"), "PIN '123' should be invalid due to insufficient digits.")
    }

    func testInvalidSequentialPin() {
        XCTAssertFalse(validator.isValid(pin: "1234"), "PIN '1234' should be invalid due to sequential digits.")
    }

    func testInvalidRepeatedPin() {
        XCTAssertFalse(validator.isValid(pin: "1111"), "PIN '1111' should be invalid due to repeated digits.")
    }
}


import SwiftUI

struct SmartPassPINView: View {
    @State private var pin: String = ""
    @State private var errorMessage: String? = nil
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Set up your Smart Pass PIN")
                .font(.title)
                .padding()
            
            SecureField("Enter PIN", text: $pin)
                .keyboardType(.numberPad)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .multilineTextAlignment(.center)
                .frame(width: 200)
                .onChange(of: pin) { newValue in
                    if newValue.count > 4 {
                        pin = String(newValue.prefix(4))
                    }
                    if !isValidPIN(newValue) {
                        pin = String(newValue.dropLast())
                    }
                }
            
            if let errorMessage = errorMessage {
                Text(errorMessage)
                    .foregroundColor(.red)
                    .font(.caption)
            }
            
            Button("Confirm PIN") {
                if isValidPIN(pin) {
                    errorMessage = nil
                    print("PIN set successfully")
                } else {
                    errorMessage = "Invalid PIN pattern. Choose a different PIN."
                }
            }
            .padding()
            .disabled(pin.count < 4)
        }
        .padding()
    }
    
    func isValidPIN(_ pin: String) -> Bool {
        guard pin.count == 4, let _ = Int(pin) else { return false }
        
        let seriesPatterns = ["1234", "2345", "3456", "4567", "5678", "6789", "0987"]
        let reversedSeriesPatterns = ["4321", "5432", "6543", "7654", "8765", "9876", "7890"]
        let repeatingNumbers = ["1111", "2222", "3333", "4444", "5555", "6666", "7777", "8888", "9999", "0000"]
        let mirroredPatterns = ["1221", "2112", "3443", "4334", "5665", "6556", "7887", "8778"]
        let repeatingDoubles = ["1212", "3434", "5656", "7878", "0909"]
        
        if seriesPatterns.contains(pin) || reversedSeriesPatterns.contains(pin) ||
            repeatingNumbers.contains(pin) || mirroredPatterns.contains(pin) ||
            repeatingDoubles.contains(pin) {
            return false
        }
        
        return true
    }
}

struct SmartPassPINView_Previews: PreviewProvider {
    static var previews: some View {
        SmartPassPINView()
    }
}




import SwiftUI

struct SmartPassPINView: View {
    @State private var pin: String = ""
    @State private var errorMessage: String? = nil

    var body: some View {
        VStack(spacing: 20) {
            Text("Set up your Smart Pass PIN")
                .font(.title)
                .padding()

            SecureField("Enter PIN", text: $pin)
                .keyboardType(.numberPad)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .multilineTextAlignment(.center)
                .frame(width: 200)
                .onChange(of: pin) { newValue in
                    let filtered = newValue.filter { "0123456789".contains($0) }
                    if filtered.count <= 4 && isValidPIN(filtered) {
                        pin = filtered
                        errorMessage = nil
                    } else {
                        pin = String(pin.prefix(filtered.count - 1))
                        errorMessage = "Invalid PIN pattern. Choose a different PIN."
                    }
                }

            if let errorMessage = errorMessage {
                Text(errorMessage)
                    .foregroundColor(.red)
                    .font(.caption)
            }

            Button("Confirm PIN") {
                print("PIN set successfully")
            }
            .padding()
            .disabled(pin.count < 4)
        }
        .padding()
    }

    func isValidPIN(_ pin: String) -> Bool {
        guard pin.count == 4 else { return false }

        let seriesPatterns = ["1234", "2345", "3456", "4567", "5678", "6789", "7890", "0987"]
        let reversedSeriesPatterns = seriesPatterns.map { String($0.reversed()) }
        let repeatingNumbers = (0...9).map { String(repeating: "\($0)", count: 4) }
        let mirroredPatterns = ["1221", "2112", "3443", "4334", "5665", "6556", "7887", "8778"]
        let repeatingDoubles = ["1212", "3434", "5656", "7878", "0909"]

        let invalidPatterns = seriesPatterns + reversedSeriesPatterns + repeatingNumbers + mirroredPatterns + repeatingDoubles

        return !invalidPatterns.contains(pin)
    }
}

struct SmartPassPINView_Previews: PreviewProvider {
    static var previews: some View {
        SmartPassPINView()
    }
}
