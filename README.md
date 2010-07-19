Tropo Gateway Module for the Drupal SMS Framework
=================================================
_Supports sending SMS, IMs, and phone calls to Drupal users as well as incoming textual messages._
--------------------------------------------------------------------------------------------------

* * *

## Quick Start
_coming soon_

## In-depth Directions
1. Sign up for a Tropo.com account (FREE) and create a new "Tropo Scripting" application.
	* Name the app anything you'd like and do one of the following:
		* Tropo hosts your Tropo app
			1. Create a new "Hosted File"
			2. Name the file anything.php and place the contents of tropo_scripting_app.php in the text box.
			3. Change the fourth line of the text box to reflect your domain/path to Drupal. Keep the ?q=.. part intact.
			4. Create the file and then application by clicking the "Create" links.
		* Self-hosted Tropo Scripting app
			1. Edit line 4 of `tropo_scripting_app.php` in the same folder as `sms_tropo.php`
			2. Change the fourth line of the text box to reflect your domain/path to Drupal. Keep the ?q=.. part intact.
			3. Enter the following URL into the "What URL powers your app?" field: `http://DRUPAL_INSTALLATION/?tropo-engine=php&q=sms/tropo/scripting_app.php`
			4. Click "Create Application"
	* Now, you will want to "Add a new phone number" as well as configure your instant messaging networks if you want. Be sure to use a dedicated screen name and not your personal one. Tropo will use these to login and send/receive message to/from
	* "Add a new token" (for both messaging and voice if you intend to let users choose voice calls for notifications instead of only messaging)
	* "Update application" to save your changes.
	
2. Install and enable the required modules:  
	_You only need to download the modules which are linked to. The others are included within them._
	* [Messaging Framework](http://drupal.org/project/messaging "Messaging Framework")
		* Messaging
		* SMS Messaging
	* [Notifications](http://drupal.org/project/notifications "Notifications")
		* Notifications
		* Content Notifications
		* Notifications UI (recommended)
		* Any other Notification modules you want to let your users use such as: Notifications UI, Taxonomy Notifications,  Autosubscribe, etc.
	* [Token](http://drupal.org/project/token "Token")
	* [SMS Framework](http://drupal.org/project/smsframework "SMSFramework")
		* SMS Framework
		* SMS User - highly recommended
		* Send to phone - if you want to be able to let users message a blog post to themselves/a friend
		* SMS Blast - if you want to be able to message all your users at once

3. Now install the sms_tropo plugin.
	* Extract the sms_tropo tarball/zip file inside the /modules/smsframework/modules/ folder so that "sms_tropo"
	* OR `cd` to `/modules/smsframework/modules/` and `git clone git://github.com/marks/sms_tropo.git`
	
4. Follow the Drupal admin navigation or browse directly to ?q=/admin/smsframework/gateways 
	* Fill out the configuration for the Tropo gateway with the details from your new Tropo Scripting application (created in step 1).
	* After saving, set Tropo as your default gateway.

### Modifications to Drupal Modules to Support Tropo's Multiple Channels
One of the great advantages of the Tropo platform is that it makes it easy to send IMs and initiate phone calls in addition to sending off SMS messages.  
_However_, there are a two code changes (required, list below) and several views changes (optional, listed further below) that you might want to make if you plan to take advantage of Tropo's non-SMS networks.
To be clear: You can use Tropo as just a SMS provider and not need to make any of the following changes. 

1. **Code change to `/modules/messaging/messaging_sms/messaging_sms.module` (Line 44)**  
	The function should be changed to read as follows:

		/**
		 * Map user account to SMS destination (phone number) __OR__ IM address. Hence the code 8 lines down.
		 */
		function messaging_sms_user_destination($account, $message) {
		  // Check for active mobile infomation. Simply return it so that the send
		  // callback has a destination array and access everything.
		  if (!empty($account->sms_user) && $account->sms_user[0]['status'] == 2 && !empty($account->sms_user[0]['number'])) {
				$destination = $account->sms_user[0]['number'];
				// Check to see if a network is defined. If so, we need to keep track of that for sending a Tropo IM/phone call (non-SMS)
				if (!empty($account->sms_user[0]['gateway']['network'])) {
					$destination = $destination . '&network=' . $account->sms_user[0]['gateway']['network'];
				}
				return $destination;
		  }
		}

2. **Code change to `/modules/smsframework/modules/sms_user/sms_user.module` (Line 269)**:  
	Change:  

		'number'  => sms_formatter($number),
			
	to: 
		
		// No longer formatting $number into a 10-digit phone number since Tropo users may be using IM addresses with non-digit characters
			'number' => $number

3. **Non-functional changes (changes to wording of other modules' to make for a better user experience)**
	
	Look through the following files for mentions of: 'SMS', 'phone', and 'number' and change them to 'message', 'device', and 'number/address'
	* `/modules/smsframework/modules/sms_user/sms_user.module`
	* `/modules/smsframework/modules/sms_actions/sms_actions.module`
	* `/modules/messaging/messaging_sms/messaging_sms.module`