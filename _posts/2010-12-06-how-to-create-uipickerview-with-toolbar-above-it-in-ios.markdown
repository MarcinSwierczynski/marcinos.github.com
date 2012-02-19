---
date: '2010-12-06 20:07:24'
layout: post
slug: how-to-create-uipickerview-with-toolbar-above-it-in-ios
status: publish
title: How to create UIPickerView with toolbar above it in iOS
wordpress_id: '38'
categories:
- iOS SDK
- Objective C
tags:
- iOS
- iPhone
- mobile
- objective c
---

A few days ago, I had to implement a UIPickerView based select list with toolbar above it. The toolbar had to have a button to close the picker. The whole solution had to be very similar to the one used in a mobile version of Safari browser. How to do that?




The clue of the problem is contained in just two properties of [UIResponder](http://developer.apple.com/library/ios/#documentation/uikit/reference/UIResponder_Class/Reference/Reference.html) class: _inputView_ and _inputAccessoryView_. The first one contains a view which will be displayed if its owner will become a first responder. For example, for UITextFields it's a keyboard by default. The second one in turn, represents a view which will be displayed before (ie. above) the view pointed by _inputView_ property.




So now it should be quite easy:



    
{% highlight objectivec %}
    UIResponder *firstResponder; // use your future first responder
    
    UIPickerView *pickerView = [[UIPickerView alloc] init];
    pickerView.showsSelectionIndicator = YES;
    
    firstResponder.inputView = pickerView;
    [pickerView release];
    
    UIToolbar *toolbar = [[UIToolbar alloc] initWithFrame:CGRectMake(0, 0, 320, 40)];
    toolbar.barStyle = UIBarStyleBlackTranslucent;
    
    firstResponder.inputAccessoryView = toolbar;
    [toolbar release];
{% endhighlight %}




Unfortunately, it isn't. Both inputView and inputAccessoryView are readonly properties, so we've to redeclare them in a subclass:



    
{% highlight objectivec %}
    @property (readwrite, retain) UIView *inputView;
    @property (readwrite, retain) UIView *inputAccessoryView;
{% endhighlight %}    




What is the best, we aren't limited to use this feature on UITextFields only. Because of fact that _UIView_ inherits from _UIResponder_, we can attach this behaviour to all views, for example to a button or a table cell. To do that we have to override `canBecomeFirstResponder` method in our subclass. For example, the UIButton subclass implementation can look like this:

{% highlight objectivec %}
    implementation CustomButton { } //it is UIButton subclass
    
    @synthesize inputView, inputAccessoryView;
    
    - (BOOL) canBecomeFirstResponder {
        return YES;
    }
    
    - (void)dealloc {
    	[inputView release];
    	[inputAccessoryView release];
    	[super dealloc];
    }
    
    @end
{% endhighlight %}
