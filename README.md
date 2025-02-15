Feature: Device Activation Negative Scenarios

  Scenario: Continue button enabled when both fields are filled
    Given I am on the device activation screen
    When I enter a valid Subscriber ID "1234567890"
    And I enter a valid User ID "user@example.com"
    Then the Continue button should be enabled

  Scenario: Continue button disabled if any field is empty
    Given I am on the device activation screen
    When I enter a valid Subscriber ID "1234567890"
    And I leave the User ID field empty
    Then the Continue button should be disabled

  Scenario: Validate maximum length of Subscriber ID and User ID
    Given I am on the device activation screen
    When I enter a Subscriber ID with more than 20 characters
    Then I should see an error message indicating that Subscriber ID exceeds maximum length
    When I enter a User ID with more than 32 characters
    Then I should see an error message indicating that User ID exceeds maximum length

  Scenario: Required message displayed when either field is empty
    Given I am on the device activation screen
    When I leave the Subscriber ID field empty
    And I try to continue
    Then I should see an error message indicating that Subscriber ID is required
    And the Continue button should be disabled

  Scenario: Subscriber ID should contain only alphanumeric characters
    Given I am on the device activation screen
    When I enter a Subscriber ID with special characters "abc123!@#"
    Then I should see an error message indicating invalid characters in Subscriber ID

  Scenario: User ID should allow alphanumeric and specific special characters
    Given I am on the device activation screen
    When I enter a User ID with invalid special characters "user^&*()@example.com"
    Then I should see an error message indicating invalid characters in User ID


import { Given, When, Then } from "@wdio/cucumber-framework";
import { expect } from "chai";
import { deviceActivationPage } from "../pages/deviceActivationPage";

Given("I am on the device activation screen", async function () {
  // Assume this method navigates to the device activation screen
  await deviceActivationPage.navigateToDeviceActivationScreen();
});

When("I enter a valid Subscriber ID {string}", async function (subscriberId:
::contentReference[oaicite:0]{index=0}
 



    page obeject ; 

import { driver } from "../utils/driver";

class DeviceActivationPage {
  // Selectors for UI elements
  private subscriberIdInput = "//XCUIElementTypeTextField[@name='Subscriber ID']";
  private userIdInput = "//XCUIElementTypeTextField[@name='User ID']";
  private continueButton = "//XCUIElementTypeButton[@name='Continue']";
  private errorMessage = "//XCUIElementTypeStaticText[@name='Error Message']";

  // Method: Enter Subscriber ID
  async enterSubscriberId(subscriberId: string) {
    const subscriberField = await driver.$(this.subscriberIdInput);
    await subscriberField.setValue(subscriberId);
  }

  // Method: Enter User ID
  async enterUserId(userId: string) {
    const userIdField = await driver.$(this.userIdInput);
    await userIdField.setValue(userId);
  }

  // Method: Click Continue Button
  async clickContinueButton() {
    const continueBtn = await driver.$(this.continueButton);
    await continueBtn.click();
  }

  // Method: Get error message text
  async getErrorMessage(): Promise<string> {
    const errorMsg = await driver.$(this.errorMessage);
    return await errorMsg.getText();
  }

  // Method: Check if Continue button is enabled
  async isContinueButtonEnabled(): Promise<boolean> {
    const button = await driver.$(this.continueButton);
    return await button.isEnabled();
  }

  // Method: Get Subscriber ID field value length
  async getSubscriberIdLength(): Promise<number> {
    const subscriberField = await driver.$(this.subscriberIdInput);
    const value = await subscriberField.getValue();
    return value.length;
  }

  // Method: Get User ID field value length
  async getUserIdLength(): Promise<number> {
    const userIdField = await driver.$(this.userIdInput);
    const value = await userIdField.getValue();
    return value.length;
  }

  // Method: Wait for a specific error message
  async waitForErrorMessage(expectedMessage: string) {
    await driver.waitUntil(
      async () => (await this.getErrorMessage()) === expectedMessage,
      {
        timeout: 5000,
        timeoutMsg: `Expected error message "${expectedMessage}" not displayed`,
      }
    );
  }
}

export const deviceActivationPage = new DeviceActivationPage();

