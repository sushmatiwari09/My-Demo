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

struct PinValidationRules {
    func isValid(pin: String) -> Bool {
        guard pin.count == 4, pin.allSatisfy(\.isNumber) else { return false }
        
        let forbiddenPatterns = generateForbiddenPatterns()
        
        return !forbiddenPatterns.contains(pin)
    }
    
    private func generateForbiddenPatterns() -> Set<String> {
        var patterns = Set<String>()
        
        // Sequential numbers (e.g., 1234, 2345)
        for i in 0...6 {
            let sequence = (i...(i+3)).map { "\($0)" }.joined()
            patterns.insert(sequence)
        }
        
        // Reversed sequential numbers (e.g., 4321, 5432)
        for i in (3...9).reversed() {
            let sequence = (i-3...i).reversed().map { "\($0)" }.joined()
            patterns.insert(sequence)
        }
        
        // Repeated numbers (e.g., 1111, 2222)
        for i in 0...9 {
            let repeated = String(repeating: "\(i)", count: 4)
            patterns.insert(repeated)
        }
        
        // Mirrored numbers (e.g., 1221, 3443)
        for i in 0...9 {
            for j in 0...9 where i != j {
                let mirrored = "\(i)\(j)\(j)\(i)"
                patterns.insert(mirrored)
            }
        }
        
        // Repeating doubles (e.g., 1212, 3434)
        for i in 0...9 {
            for j in 0...9 where i != j {
                let repeatingDoubles = "\(i)\(j)\(i)\(j)"
                patterns.insert(repeatingDoubles)
            }
        }
        
        return patterns
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
