#Hands-On Workshop: Write your first Apex Trigger!#

##About the Workbook##
This workbook shows you how to write a simple Apex trigger and deploy it in a production org. No prior coding experience is needed, however, a basic understanding of the Salesforce platform is helpful.

Apex triggers are used to extend the capabilities of the Salesforce platform beyond the functionality available to point-and-click admins. While workflow rules, visual flows, validation rules, and formula fields are powerful and simple to use, they aren’t able to accomplish advanced functionality available in Apex triggers, such as:

* Creating new records
* Referencing multiple related or unrelated records
* Access special information about your database, such as permissions
* Integrations with external systems

This workbook will teach you all the components needed deploy an Apex trigger:

* How to write an Apex trigger 
* How to write a test class for your trigger
* How to deploy your Apex trigger using Change Sets

Apex triggers are a fun and surprisingly easy way to unlock the potential of your Salesforce org!

###Audience###
These tutorials are intended for admins who have experience working on the Salesforce platform but no coding background.

Users should be familiar with the following basic concepts of Salesforce before delving into Apex code:

* How to create an App and Database
* Customizing the User Interface
* Add app logic using workflow rules, formula fields, and validation rules
* Analyzing data using Reports and Dashboards

We recommend referring to the Force.com Workbook to learn more about the basic Salesforce concepts listed above.

###Sign up for Developer Edition###
This workbook is designed to be used with a Sandbox or Developer Edition organization (or DE org for short). DE orgs are multipurpose environments with all of the features and permissions that allow you to develop, package, test, and install apps.

1. In your browser, go to https://developer.salesforce.com/signup.
2. Fill in the fields about you and your company.
3. In the Email Address field, make sure to use a public address you can easily check from a Web browser.
4. Type a unique Username. Note that this field is also in the form of an email address, but it does not have to be the same as your email address, and in fact, it’s usually better if they aren’t the same. Your username is your login and your identity on `developer.salesforce.com`, so you’re often better served by choosing a username such as `firstname@lastname.com`.
5. Read and then select the checkbox for the `Master Subscription Agreement` and then click `Submit Registration`.
6. In a moment you’ll receive an email with a login link. Click the link and change your password.
You will only be able to deploy your trigger to a production environment if you use Sandbox org. Code written in DE orgs cannot be deployed to production environments.

##Create an Apex trigger##
###Introduction###

We’ll be making a trigger that automatically creates a new Case when an Account is created in the database. We’ll also use values of the Account to populate fields on the Case we’re creating.

1. The basic logic will go like this:
  1. A new Account is created in Salesforce
  2. Trigger begins:
    1. A new Case is created:
    2. The Case will be associated with the Account
    3. The Subject field of the Case will be set to: "Send {!Account.Name} a free T-shirt!"
    4. The Priority field of the Case will be set to High

So when an Account is created like this:
<img src="https://lh4.googleusercontent.com/Ktr9JkX2BoOBFUCJTYV1uSDV95dGNtzErR3MosF85Vxw4a_UbWPj46A8JXrUzVmi3uMHp7EJ6wswNd_oWjPB2fenjnQ1ivjRIC-Dm5Rl2QjeEfgWMjKwGlfNp-a3nUBXew" />

The trigger will automatically create a Case like this:
<img src="https://lh6.googleusercontent.com/fgy_Y9_yL0X7KxJG0x2LCdAXURU6QLV7qyh7100diiQ47Vj_R80Q3hxVBeTxv7bzKroXQYCo2REKaYocT3-mb-H25voT7Ux89rVX45bTyQUQOxLneKR-EaNljVJOQXpQZw" />

This trigger will do something that’s currently impossible using any point-and-click functionality: automatically create new records.

The purpose of this tutorial is not to build a trigger with complex logic. This tutorial is meant to give you an introduction to coding and teach you the fundamentals of Apex triggers so you can have an understanding of what a trigger is and how it works.

###Step 1: Navigate to the trigger builder page###
1. Launch your browser and log into Salesforce.
2. Go to the trigger page of the Account object
  1. `Your Name >> Setup`
  2. `Customize >> Accounts >> Triggers`
  3. Click the `New` button to create a new trigger
3. The following page should be visible to you. A pre-made trigger template is automatically created for your convenience:
<img src="https://lh4.googleusercontent.com/pb-VyKLTt6gk_BD6iKRU97-B3aqSnwFGUQNW_z9VBD0JorxDE6f_19GTJIaVvbSd8wgQxILJNkrCYu2MlrT2xIeDqATexmHj8EqvmaPztWdukUg_ZjzcRPV2MFS2l1iMow" />

###Step 2: Describe the high-level details of your trigger###
In this step we’ll modify the pre-made trigger template for our specific needs
1. Name your trigger:
  1. Replace `<name>` with the name of our trigger: `TShirtReminder`
  2. Note that trigger names should contain only alphanumeric characters and no spaces. 
  3. Although this isn’t required, it is customary for the first letter of each word in the name to be capitalized for easy reading
2. Define the trigger's events:
  1. Replace `<events>` with `after insert`
    1. Events describe when our trigger should run. 
    2. Each event has a type and an operation.
    3. There are two types:
      * `before`
      * `after`
    4. There are six operations:
      * `update`
      * `insert`
      * `delete`
      * `merge`
      * `undelete`
      * `upsert`
    5. Any type can be combined with any operation to make an event. Multiple events can be defined in your trigger by separating them using commas, ie: `after update, after insert, before delete`
3. For this trigger, we chose `after insert` because we want this trigger to run only after an Account is created. We do not want this trigger to run for example when an Account is updated or deleted.
4. Your trigger should now look like this:
<img src="https://lh4.googleusercontent.com/guInUioNXRfI0YZad0oWAao0heGYnY5mxm3bK1quoPr6tJQVnqO13lt1fxrZ8NUUeraBRxSrEFHDpGVCyNghzAKPgEesCxnU2KwFUiN5c2XLdZwgwu7W5BeOrnUJMY3NzQ" />

###Step 3: Create a loop in your trigger###
Since our trigger runs whenever an Account is created, it’s possible that many Accounts might be created at the exact same time (for example, through a mass insert using Data Loader). We’ll use a loop to repeat our trigger’s logic for each record in our trigger.

1. Create a for loop to repeat logic among multiple records.
2. Add the following lines of code in between your trigger's brackets:

```
for (Account acc : Trigger.new) {

}
```

  * How this loop works:
    * In human terms, the loop says "for every Account in our trigger, repeat all the code within these brackets".
    * A for loop does the following:
      1. Creates a variable that represents one record out of a list.
      2. Iterates across every record in the list, assigning it to the variable.
      3. Repeats all the code within its brackets for each record.
        * `acc` is a variable we're assigning to every Account going through our trigger. We precede it with Account because the variable `acc` is always an Account.
        * `Trigger.new` is the list of every account going through our trigger. Since our trigger runs after insert, `Trigger.new` is every account currently being inserted, which could be one or many accounts.
3. Your trigger should now look like this:
<img src="https://lh3.googleusercontent.com/HYDDVXpiCn-j3EDcYmZeJHSCcAZBhhG6dXREkEjnyQHjs-UfwzWNkSTnt5P6OkmM54rbXA9s9kw-6vThMhnM2w_3wUwS2t9S5BJM8BqoYlgMCu27ozW9F7mSjxbORYSlhA" />

Every trigger follows the same general pattern you see above!

###Step 4: Add business logic to your trigger###
In this step we’ll write the code that creates a Case and associates it to our Account.

1. Create a new Case:
```Case shirtCase = new Case();```
  * We create a new Case and assign it to a shirtCase variable.
  * The objects here can be substituted by any other SObject (ex: a Task).
2. Populate the fields of the Case:
```
shirtCase.Subject   = 'Send ' + acc.Name + ' a free t-shirt!';
shirtCase.Priority  = 'High';
shirtCase.AccountId = acc.Id;
```
  * We use a similar "dot notation" syntax when accessing fields via code as we would do in a formula field.
  * Text is always wrapped in single quotes.
  * `acc` is the Account variable of the current Account we're iterating through.
3. Insert the case into the database:
```insert shirtCase;```
  * This line of code saves the Case to the database.
4. Your trigger should now look like this:
<img src="https://lh4.googleusercontent.com/V_oBGgRJlEafrHmtqS74m0LRoD08eMBu4HwDIwDI1Lxd_pNMU4RSYwEVMhPzEkUm1wylJtFEXlvmPbwvKCZFXyjgapDp18Cc11UicpiMU07uzhyHWHzia2dw2YC2s1-jrg" />

###Step 5: Check if your trigger works!###
Your trigger is now active in your org and ready for testing!

1. Go to the Accounts tab.
2. Create a new Account with name `Salesforce University`
3. Save your Account.
4. Check to see if a Case was created on the Account!
<img src="https://lh6.googleusercontent.com/fSPO4z8jwPzZusnk1-yju1Xk6Ip_TOTvbIDrmL-3T-pN8czOjJWJf5PDCGISvhSCvwZZkgae-gYL77KIVf6qm_uEWvB0AX58xht_4PnnLzPh67r0jdg5KNyMRDvmYBXICw" />

##Create a Test Class##
###Introduction###
In this section we'll create a test class for our `TShirtReminder` trigger.

Test classes are a best practice in any programming language. Since Salesforce is a cloud platform, test classes are required to prevent organizations from deploying code that doesn't work.

Even though our trigger works in our Sandbox/DE org, we won’t be able to deploy it to a production org unless we have a test class.

###How test classes work###
75% is the magic number for passing Salesforce's code coverage requirement.

That means that our test class needs to run at least 75% of the code we wrote in our trigger before we’re able to deploy it to a production environment. 

Remember when we manually checked to see if our trigger worked in step 5 of the previous section? We basically want to do the exact same thing in our test class, but programmatically:

1. Create a new Account and save it.
2. Check to see if a Case was automatically created on the Account.
3. Make sure the Case was created with the proper fields populated.

###Step 1: Set up your test class###
In this step we’ll set up our general test class template.

1. Navigate to the Apex class editor page:
2. `Your Name >> Setup`
3. `Develop >> Apex Classes`
4. Hit the `New` button to create a new Apex class.
5. Type the following code into the code editor:
```
@isTest
public class TestTShirtReminder {
	static testmethod void createAccount() {

	}
}
```
  * What you're doing:
  * The `@isTest` lets Salesforce know that you're writing a test class. This helps them determine if you'll be able to deploy code.
  * `TestTShirtReminder` is the name of our test class.
  * `createAccount` is the name of our test method that will run when Salesforce is testing your code.
  * Don't worry about what the rest of the code says, every test class uses the same language so you'll simply copy/paste that syntax in the future.
6. Your test class should look like this so far:

<img src="https://lh5.googleusercontent.com/M6h8bgXvfjtBT_r3366x8R46hl-OHl8pOwc7EUs3hklzLfoPsQ6Gj-_BqoNWmt_oMXmz8C7q9PaUds4YpDPoMn0QX_VTt5vRr_aFQ4L5h2s16aI5x_PuJ0a9mAwcOa_u8g" />

###Step 2: Add business logic to your test class###
We'll add business logic in our code that will make at least 75% of the code in our trigger run. Since our trigger runs whenever a new Account is created, we simply need to create a new Account and 100% of the lines of code in our trigger will run.

1. Create a new Account
2. Add the following lines of code inside the `createAccount()` method brackets:
```
Account acc = new Account();
```
  * This creates a new Account and sets it to an `acc` variable 
```
acc.Name    = 'Salesforce University';
```
  * This sets the Account name to Salesforce University
```
insert acc;
```
  * This line saves the Account into the Salesforce database
3. Your code should now look like this:
<img src="https://lh4.googleusercontent.com/BzlBYmAwWmx1M7N4xrBwM-FaOMJ0bo-jibWg1bBPgBOTSQmkn0KZ-2Cqpw9GiPsZARv4uLWsl6zM9lNFo4uh6fDtg--se901zVq9AM0cf97Da75FMBjuJkftW44NTAHDHQ" />


###Step 3 (Optional): Double check that the Case was created properly###
We'll be using some more advanced Apex in this step to programmatically ensure our Case was properly created. This is a testing best practice but not a requirement for deploying.

1. Use SOQL to query for the created Case:
  * Add the following line of code to the test class:
```
Case c = [SELECT Id, Subject, Priority FROM Case WHERE AccountId = :acc.Id];
```
  * We're querying for the Case and all the fields our `TShirtReminder` trigger is supposed to be modifying, then assigning that Case to a variable `c`
  * We have a filter in our SOQL query that only retrieves a Case if it's associated with the Account we created in Step 2. 
2. Assert that the fields on the Case were properly set:
  * Add the following lines of code to the test class:
```
System.assertEquals('Send Salesforce University a free t-shirt!', c.Subject);
System.assertEquals('High', c.Priority);
```
  * `System.assertEquals()` is a method that takes two inputs and makes sure they're exactly the same.
  * The first input is what we expect the value to be, and the second input is the actual value.
  * If the two values are the same, our test class will generate an error message.
3. Your code should now look like this:
<img src="https://lh6.googleusercontent.com/-n37wjoTxsf5kfc2j1beXtQgY8SDucZO9AQgftpuKBaVkXUd00nveZd4fx1Nu-AEajKdF_WKVfAp_iubbxEOXqJ9FdNZzcZ1LK6EDThLQV-G1i6Dp3Q3pI3N4kz6NdFZpw" />


###Step 4: Run your test class###
Let's run our test class and make sure we have the required code coverage.

1. Run your test class:
  1. Save your test class and hit the `Run Test` button.
  2. A green check box will appear after a short processing time, indicating that no errors were found.
2. Check our trigger code coverage percentage:
  1. Navigate back to our `TShirtReminder` trigger:
  2. `Your Name >> Setup`
  3. `Develop >> Apex Triggers >> TShirtReminder`
  4. The `Code Coverage` field should read `100% (6/6)`
3. Your trigger is now ready to be deployed to a production environment!
<img src="https://lh4.googleusercontent.com/Et_FQHJG7sQgtOSfJUEZuyGENyu9pqsP5XLhtR_OnNmPusDD38NTWEKeFWwIUyvsvyNI_LO17tIykluqizpjZvuRFZazE7QzJiEVBD8GQkAOULgi_ZFYPyVZRQVrgZBsQw" />


##Deploying your Code##
###Introduction###
The final piece is to deploy your code into a production environment. 

The deployment process migrates your code from a Sandbox environment into your production Salesforce org. During the deployment phase, Salesforce checks your test code coverage to see if you've reached a minimum of 75% overall code coverage. If so, your code will be immediately pushed to production. If not, your deployment will fail until you modify your test classes to reach the 75% code coverage requirement.

Note that code deployed to a production environment must be deployed from a Sandbox environment - DE orgs cannot push code to production.

There are multiple ways to deploy code. We’ll be using a feature called Change Sets, which allows you to deploy code using a simple point-and-click interface. 

###Step 1: Create a Deployment Connection###
Before we can deploy using Change Sets, we must make sure our Sandbox org is connected to a production environment. This is a one-time setup step for deploying any code from your Sandbox to production orgs. Future code deployments will be pre-configured as a result of this task.

1. Create a deployment connection from your production org:
  1. Login to your production org.
  2. `Your name >> Setup`
  3. `Deploy >> Deployment Connections`
  4. Click on the Sandbox you'd like to deploy code from.
  5. Check the `Allow Inbound Changes` checkbox.
<img src="https://lh6.googleusercontent.com/SaqxI6KK5I9fqc8fu3xBPtoyACIOOfWVELvZ8LSxsxrBIiBKvWlrrxbShE9KFXtjryOjStxatDKuPs5hQg2xQCB7OQSoRQ9aKyxONhL_y60FQuICX0RsAkbl3Si1qCUkRQ" />
2. Confirm that your Sandbox org is configured to deploy to your production org:
  1. Login to your Sandbox org.
  2. `Your name >> Setup`
  3. `Deploy >> Deployment Connections`
  4. Click on the production org.
  5. Ensure that the `Accept Outbound Changes` checkbox is checked.
<img src="https://lh3.googleusercontent.com/nNUOUWlusq0NLQ8Z71oYOVpj3gClTCIbpi_nXkLD4C9RSVMRxMfYdHfwLqSbOl8txwRsMcTojU4XSmw_PXVKyhmyqax0RgHOpZe6a6nh7DaGviOTJ8PHBoAeMDGeHAsRmg" />
Congratulations! You’re now ready to deploy code!

###Step 1: Create a Change Set###
A Change Set is a package of components that you want to move from one org to another. In this scenario we'll be adding our trigger and test class to our Change Set, however you can also package other components such as custom fields, workflow rules, validations, etc.

1. Navigate to the Change Set wizard in your Sandbox environment:
  1. `Your name >> Setup`
  2. `Deploy >> Outbound Change Sets`
  3. Hit the `New` button to create a new Change Set.
    * Name: `T-Shirt Trigger and Test Class`
    * Description: `Our first Change Set!`
  4. Save the Change Set.
2. Your Change Set should now look like this:
<img src="https://lh6.googleusercontent.com/k3uGpUdrcWAnIEE34Iq-9b9T7sMmDtEipkn1FapQ93mcNl6ud-dqgUUQ_g7TsFvZBNqox_Hbh8smDLaujWVnGIHD2g39FY-zjOJtweX31COfmnOQzDBFqLqSTqOb6hh05w" />


###Step 2: Add our Trigger and Test Class to the Change Set###
We'll add both our trigger and our test class to the Change Set. Note that both should be deployed at the same time to meet the test code coverage requirements.
1. Add the `TShirtReminder` trigger to our Change Set.
  1. Click the `Add` button to add a component to the Change Set.
  2. Change the Component Type picklist to `Apex Trigger`.
  3. Check the `TShirtReminder` trigger and add it to the Change Set.
2. Add the `TestTShirtReminder` test class to our Change Set.
  1. Click the `Add` button to add a component to the Change Set.
  2. Change the Component Type picklist to `Apex Class`.
  3. Check the `TestTShirtReminder` Apex Class and add it to the Change Set.
3. Your Change Set should now look like this:
<img src="https://lh3.googleusercontent.com/GCMNF3emtCodzd9I3n7JkkxFyoRYa7Ir1wHaKXTlQVvVR4umjAQyhL1GOyt5WieQQ9NSNHZ9iaQr6QnbZKVxCRFYNZveI7lV16VH6JqatZqtQbTGFi5GS2Wiv86yaRn4qw" />


###Step 3: Deploy our Change Set###
Our Change Set is now ready to be deployed to a production org.

1. Upload the Change Set:
  1. Hit the `Upload` button.
  2. Select the `Production` org.
  3. Click the` Upload` button.
<img src="https://lh4.googleusercontent.com/u5Jojob6HuqHLWR44-uFbPXi4pmAH4UEEaNXOmduZBcJf826OqzAAKgm2JiAm98sUBzpH0zom5WiKai5VnHYT1Ti9-e74LWW0Brpdy_6UZdGy4rwi2VOMNSZxreG99y_Mg" />
2. Accept the Change Set in your production org:
  1.Login to your production org.
  2. `Your Name >> Setup`
  3. `Deploy >> Inbound Change Sets`
  4. Click the `T-Shirt Trigger and Test Class` Change Set.
    * Note that it can take up to 30 minutes for your inbound Change Set to appear in your production org. Most of the times it takes no more than a few minutes.
  5. Click the `Deploy` button.
<img src="https://lh6.googleusercontent.com/IqfIs2f3RaMpXMGXJj1nBUsOJeaLKkHwm-UsfTUE-7DTxDQKAZ84cD6NyXpHqJkx967jTnYaZqnaVxUrSlwwwQuJH5x4XY-rjWYNAh6g6WfBUdZOs-b-4jiX4T3Qkl9sBQ" />

##Congratulations, your code is now deployed in your production org!##
