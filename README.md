@demo
Feature: Login to the application

  Scenario: Continue button disabled if any field is empty
    Given I am on the device activation screen
    When I enter a valid Subscriber ID "123456sdfdfg7890" and leave the User ID field empty
    Then I should see an error message indicating that Subscriber ID is required And the Continue button should be disabled

  Scenario: Validate maximum length of Subscriber ID and User ID
    Given I am on the device activation screen
    When I enter a Subscriber ID with more than 20 characters
    Then I should not be able to type any characters
    When I enter a User ID with more than 32 characters
    Then I should not be able to type any characters

  Scenario: Required message displayed when either field is empty
    Given I am on the device activation screen
    When I leave the Subscriber ID field empty
    Then I should see an error message indicating that Subscriber ID is required And the Continue button should be disabled

  Scenario: Subscriber ID should contain only alphanumeric characters
    Given I am on the device activation screen
    When I enter a Subscriber ID with special characters "abc123!@#"
    Then I should not be able to type any special characters

  Scenario: User ID should allow alphanumeric and specific special characters
    Given I am on the device activation screen
    When I enter a User ID with invalid special characters "user&%()example.com"
    Then I should not be able to type any special characters which is invalid

  Scenario: Validate empty spaces in Subscriber ID and User ID fields
    Given I am on the device activation screen
    When I enter spaces in the Subscriber ID or User ID fields
    Then I should see an error message indicating that spaces are not allowed

  Scenario: Validate minimum length requirement for Subscriber ID and User ID
    Given I am on the device activation screen
    When I enter a Subscriber ID with fewer than 5 characters
    Then I should see an error message indicating that the Subscriber ID must have a minimum of 5 characters
    When I enter a User ID with fewer than 5 characters
    Then I should see an error message indicating that the User ID must have a minimum of 5 characters

  Scenario: Validate case sensitivity in User ID
    Given I am on the device activation screen
    When I enter a User ID with mixed uppercase and lowercase characters "TestUser123"
    Then The system should accept the User ID and retain the case

  Scenario: Validate Subscriber ID and User ID against SQL injection
    Given I am on the device activation screen
    When I enter an SQL injection attempt "' OR '1'='1"
    Then The system should reject the input and show an error message

  Scenario: Validate XSS attack prevention in Subscriber ID and User ID
    Given I am on the device activation screen
    When I enter a script tag "<script>alert('XSS')</script>" in the Subscriber ID or User ID field
    Then The system should sanitize the input and not execute any scripts

  Scenario Outline: As a user, I can register into the application using subId and userId
    Given I am on the device activation screen
    When I enter the "<subId>" and "<userId>"
    Then I should be successfully registered

    Examples:
      | subId      | userId        |
      | user12345  | testuser1     |
      | test123    | testUser2     |
      | sub56789   | demoUser123   |




import { $, browser, by, element } from 'protractor';

export class DeviceActivationPage {
    // Locators
    subscriberIDField = $('#subscriberID');
    userIDField = $('#userID');
    continueButton = $('#continue');
    errorMessage = $('.error-message');

    // Methods
    async navigateTo() {
        await browser.get('/device-activation');
    }

    async enterSubscriberID(subscriberID: string) {
        await this.subscriberIDField.clear();
        await this.subscriberIDField.sendKeys(subscriberID);
    }

    async enterUserID(userID: string) {
        await this.userIDField.clear();
        await this.userIDField.sendKeys(userID);
    }

    async clickContinue() {
        await this.continueButton.click();
    }

    async getErrorMessage() {
        return await this.errorMessage.getText();
    }

    async isContinueButtonDisabled() {
        return !(await this.continueButton.isEnabled());
    }
}


import { Given, When, Then } from 'cucumber';
import { expect } from 'chai';
import { DeviceActivationPage } from '../pages/deviceActivation.page';

const deviceActivationPage = new DeviceActivationPage();

Given('I am on the device activation screen', async () => {
    await deviceActivationPage.navigateTo();
});

When('I enter a valid Subscriber ID {string} and leave the User ID field empty', async (subscriberID: string) => {
    await deviceActivationPage.enterSubscriberID(subscriberID);
});

Then('I should see an error message indicating that Subscriber ID is required And the Continue button should be disabled', async () => {
    const message = await deviceActivationPage.getErrorMessage();
    expect(message).to.include('Subscriber ID is required');
    expect(await deviceActivationPage.isContinueButtonDisabled()).to.be.true;
});

When('I enter a Subscriber ID with more than 20 characters', async () => {
    await deviceActivationPage.enterSubscriberID('123456789012345678901');
});

Then('I should not be able to type any characters', async () => {
    const value = await deviceActivationPage.subscriberIDField.getAttribute('value');
    expect(value.length).to.be.at.most(20);
});

When('I enter a User ID with more than 32 characters', async () => {
    await deviceActivationPage.enterUserID('user_with_more_than_32_characters_example');
});

Then('I should not be able to type any characters', async () => {
    const value = await deviceActivationPage.userIDField.getAttribute('value');
    expect(value.length).to.be.at.most(32);
});

When('I enter a Subscriber ID with special characters {string}', async (subscriberID: string) => {
    await deviceActivationPage.enterSubscriberID(subscriberID);
});

Then('I should not be able to type any special characters', async () => {
    const value = await deviceActivationPage.subscriberIDField.getAttribute('value');
    expect(value).to.match(/^[a-zA-Z0-9]*$/);
});

When('I enter an SQL injection attempt {string}', async (sqlInjection: string) => {
    await deviceActivationPage.enterSubscriberID(sqlInjection);
});

Then('The system should reject the input and show an error message', async () => {
    const message = await deviceActivationPage.getErrorMessage();
    expect(message).to.include('Invalid input');
});

When('I enter a script tag {string} in the Subscriber ID or User ID field', async (scriptTag: string) => {
    await deviceActivationPage.enterSubscriberID(scriptTag);
});

Then('The system should sanitize the input and not execute any scripts', async () => {
    const value = await deviceActivationPage.subscriberIDField.getAttribute('value');
    expect(value).not.to.include('<script>');
});
