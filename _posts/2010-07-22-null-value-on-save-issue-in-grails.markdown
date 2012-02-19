---
date: '2010-07-22 22:07:11'
layout: post
slug: null-value-on-save-issue-in-grails
status: publish
title: Null Value on Save Issue in Grails
wordpress_id: '9'
categories:
- Grails
- Groovy
- Programming
tags:
- english
- grails
- groovy
- programming
---

If you’ve used Grails, you’re probably familiar with a domain class and its “constraints” block. There you can define conditions which have to be met by class’s fields. For example, you can enforce that a field have to be not-empty using “blank: false” condition. It’d be intuitive not to define conditions for the field you don’t care of. Unfortunately, there is a small trap here. Let’s use an example to explain it.





Let’s define an Invoice class with a few fields: number, draw date and payment date. The number and draw data are obligatory while the payment date isn’t, because you’re able to pay for an invoice by cash.




    
    
    class Invoice { 
    	String number; 
    	Date drawDate; 
    	Date paymentDate; 
    	static constraints = { 
    		number(blank: false); 
    		drawDate(blank: false); 
    	}
    } 
    





It looks good, doesn’t it? You can generate controller and views for that class and test it in a browser. Try to leave a payment date field empty and save an invoice. Success – it works! OK, so let’s create a BootStrap entry for our new class. Again, try to omit its optional field – paymentDate.




    
    
    def invoice = new Invoice(number: “1/2010”, drawDate: new Date()).save(); 
    





What value does invoice variable have? Null! What’s wrong? Why does it work in a browser and not directly in a code?





The answer is quite straight, <del>but I haven’t found it in a documentation</del>. The default, implicit value of a field’s constraint is “nullable: false”. When you fills in a form in your browser, you really sends a blank value – empty string. This string isn’t null so it meets “nullable: false” criteria. On the other hand, if you creates a new object in your code and you omits a field, you pass a null value and hence the validator doesn’t allow to create an object! Unfortunately, Grails doesn’t provide any descriptive message on what’s really going under the hood.





What can you do? You can set an explicit constraint for such a field: “nullable: true”. The complete class would be:




    
    
    class Invoice { 
    	String number; 
    	Date drawDate; 
    	Date paymentDate; 
    	static constraints = { 
    		number(blank: false); 
    		drawDate(blank: false); 
    		paymentDate(nullable: true);
    	} 
    }
    





Personally, I suggest to set “nullable: true” constraint for every field which is optional and “blank: false” for every obligatory field.



This article was previously posted at [Groovy Zone](http://groovy.dzone.com/tips/null-value-save-issue-grails).
