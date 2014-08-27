---
layout: post
title: How I transfered my Play Store apps to another account and refunded developer registration fee
tags:
- android
- google
published: true
---

Yesterday I moved all of my Play Store apps to a new account due to some issues on my Google Wallet.

And here I record the process of transfer and refund:

----

## 1. Create a new Google account.


----

## 2. Finish the developer registration process on that new account.

If you proceed to the next step without finishing the registration process, you\'ll receive an email like this:

	After researching the target account, we've found that the registration fee payment is still in process.
	This can take up to 48 hours to complete. 
	
	If the account owner for the target account still sees the 'Your registration is being processed...' banner
	in your Google Play Developer Console after 48 hours from their registration,
	please have them contact the Google Wallet team at: 
	https://support.google.com/wallet/#topic=3209987&contact=1

	Once the target account registration has fully processed,
	feel free to respond to this email and we'll proceed with your app transfer. 

----

## 3. Request for a transfer.

### A. Get the transaction ids of developer registration fee. (both the old and new account)

Visit [Google Wallet](http://wallet.google.com/manage) page of each account, and find the transactions of your developer registration fee.

![google_wallet_transactions](https://cloud.githubusercontent.com/assets/185988/3806984/c79f6842-1c5a-11e4-9c3a-d6d868091779.png)

You can find the transaction ids in them.

![google_wallet_transaction_id](https://cloud.githubusercontent.com/assets/185988/3806985/c79f69aa-1c5a-11e4-8ce5-3db79ad209e9.png)

### B. Request a transfer.

Go to [this link](https://support.google.com/googleplay/android-developer/checklist/3294213) and fill out the form.

You\'ll also have to write down the titles and package names of your apps.

The request will be processed in 48 hours(or even faster if you are lucky) after the submission.

----

## 4. Request for a refund of your old account\'s developer registration fee.

### A. Make sure you have no apps left on your old account.

If you go to the next step with apps left on your account, you\'ll receive an email like this:

	Unfortunately we are unable to close a Google Play Developer Console account that has apps with customer downloads.
	As you may recall, our Developer Distribution Agreement (DDA) allows Google Play customers unlimited app reinstalls.
	For questions on our DDA, please follow this link: http://www.android.com/us/developer-distribution-agreement.html
	
	We would like to offer a solution, however.
	If you are able to find another Android developer to buy or otherwise receive your apps,
	we will be happy to transfer the apps to the another Developer Console account.
	Once your Developer Console account no longer has any apps associated with it, we can close the account and issue you a refund.

So check your old account out if the developer console is empty.

### B. Request a refund.

Visit [this link](https://support.google.com/googleplay/android-developer/contact/dev_registration?extra.IssueType=cancel) and submit the form.

It will also be processed in 48 hours(or less).

----

## 5. (Optional) Delete your old account.

Make sure you\'re signed out from your new account and sign in back with your old account.

And do as [this link](https://support.google.com/accounts/answer/32046) says.

