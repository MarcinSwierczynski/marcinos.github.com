---
date: 2010-07-25 16:15:19 +0100
layout: post
slug: how-to-add-a-text-message-to-messages-conversation-in-android-sdk
status: publish
title: How to add a text message to messages conversation in Android SDK
wordpress_id: '16'
categories:
- Android SDK
- Java
- Programming
tags:
- android
- autoresponder
- english
- java
- programming
---

Today I've published new version of my Andriod application - [AutoRespnder](http://autoresponder.swierczynski.net/). The main feature in this release was quite simple: show auto-sent messages within standard messages conversation, just as if was send by hand. How to do this in Android SDK?




Because it is a background operation not visible to user, I've decided to build a `Service` to accomplish this task.




```java
    //imports

    public class SentSmsLogger extends Service {

    	private static final String TELEPHON_NUMBER_FIELD_NAME = "address";
    	private static final String MESSAGE_BODY_FIELD_NAME = "body";
    	private static final Uri SENT_MSGS_CONTET_PROVIDER = Uri.parse("content://sms/sent");

    	@Override
    	public void onStart(Intent intent, int startId) {
    		addMessageToSentIfPossible(intent);
    		stopSelf();
    	}

    	private void addMessageToSentIfPossible(Intent intent) {
    		if (intent != null) {
    			String telNumber = intent.getStringExtra("telNumber");
    			String messageBody = intent.getStringExtra("messageBody");
    			if (telNumber != null && messageBody != null) {
    				addMessageToSent(telNumber, messageBody);
    			}
    		}
    	}

    	private void addMessageToSent(String telNumber, String messageBody) {
    		ContentValues sentSms = new ContentValues();
    		sentSms.put(TELEPHON_NUMBER_FIELD_NAME, telNumber);
    		sentSms.put(MESSAGE_BODY_FIELD_NAME, messageBody);

    		ContentResolver contentResolver = getContentResolver();
    		contentResolver.insert(SENT_MSGS_CONTET_PROVIDER, sentSms);
    	}

    	@Override
    	public IBinder onBind(Intent intent) {
    		return null;
    	}

    }
```




`SentSmsLogger` expects `Intent` with receiver number and message body. Then it passes that information to proper `ContetProvider`.




And this is the clue - it isn't well documented how to manage `ContentProvider` associated with messaging module. Google to the rescue ;) I've found an information that relevant `ContentProvider` has URI `content://sms/sent`. Next step was to find out what are the names of fields that contains data about message body and its receiver. With the help of debugger, I've found them - it's `body` and `address`. I've put all of this in private constants in the top of class.




That's all! Now we've to use this data in a standard manner. Useful information about that can be found [in official documentation](http://developer.android.com/guide/topics/providers/content-providers.html).




**Important note**: due to compatibility issues, I've used Android SDK v. 1.5 here.
