# My-Demo

Feature: Device Activation Negative Scenarios

  Scenario: Trying to proceed without entering User ID
    Given I am on the device activation screen
    When I leave the User ID field empty
    And I try to continue
    Then I should see an error message indicating that User ID is required

  Scenario: Entering an invalid User ID format
    Given I am on the device activation screen
    When I enter an invalid User ID "abcd@123"
    And I try to continue
    Then I should see an error message indicating that the User ID format is incorrect

  Scenario: Entering an incorrect Subscriber ID
    Given I am on the device activation screen
    When I enter an incorrect Subscriber ID "000000"
    And I try to continue
    Then I should see an error message indicating that the Subscriber ID is invalid

  Scenario: Leaving both fields empty
    Given I am on the device activation screen
    When I leave the Subscriber ID and User ID fields empty
    And I try to continue
    Then I should see an error message indicating that both fields are required

  Scenario: Checking if the "Continue" button is disabled with incomplete input
    Given I am on the device activation screen
    When I enter only the Subscriber ID "4567465"
    And I do not enter the User ID
    Then the Continue button should be disabled



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

  // Method: Check if Continue button is disabled
  async isContinueButtonDisabled(): Promise<boolean> {
    const button = await driver.$(this.continueButton);
    return !(await button.isEnabled());
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


import { Given, When, Then } from "@wdio/cucumber-framework";
import { expect } from "chai";
import { deviceActivationPage } from "../pages/deviceActivationPage";

Given("I am on the device activation screen", async function () {
  await deviceActivationPage.waitForDeviceActivationScreen();
});

When("I leave the User ID field empty", async function () {
  await deviceActivationPage.enterUserId("");
});

When("I enter an invalid User ID {string}", async function (userId: string) {
  await deviceActivationPage.enterUserId(userId);
});

When("I enter an incorrect Subscriber ID {string}", async function (subscriberId: string) {
  await deviceActivationPage.enterSubscriberId(subscriberId);
});

When("I leave the Subscriber ID and User ID fields empty", async function () {
  await deviceActivationPage.enterSubscriberId("");
  await deviceActivationPage.enterUserId("");
});

When("I enter only the Subscriber ID {string}", async function (subscriberId: string) {
  await deviceActivationPage.enterSubscriberId(subscriberId);
});

When("I try to continue", async function () {
  await deviceActivationPage.clickContinueButton();
});

Then(
  "I should see an error message indicating that User ID is required",
  async function () {
    await deviceActivationPage.waitForErrorMessage("User ID is required.");
  }
);

Then(
  "I should see an error message indicating that the User ID format is incorrect",
  async function () {
    await deviceActivationPage.waitForErrorMessage("Invalid User ID format.");
  }
);

Then(
  "I should see an error message indicating that the Subscriber ID is invalid",
  async function () {
    await deviceActivationPage.waitForErrorMessage("Invalid Subscriber ID.");
  }
);

Then(
  "I should see an error message indicating that both fields are required",
  async function () {
    await deviceActivationPage.waitForErrorMessage("Subscriber ID and User ID are required.");
  }
);

Then("the Continue button should be disabled", async function () {
  const isDisabled = await deviceActivationPage.isContinueButtonDisabled();
  expect(isDisabled).to.be.true;
});


