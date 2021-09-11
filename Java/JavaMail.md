## JavaMail Best Practices[1]

* JavaMail service providers divide the messaging world into stores, which hold incoming messages, and transports, which launch messages toward a destination. 
* `Message` objects are used to represent individual emails.
* A `Session` object is used to tie `message stores`, `transports`, and `messages` together
* The standard JavaMail implementation from Sun provides for `POP3 and IMAP message stores`, and for `SMTP message transport`.

### Understanding Enterprise Email
* RFC 822
* MIME(Multimedia Internet Mail Extensions) gave email messages the ability to exchange any kind of content, rather than just text, by standardizing the schemes for encoding bytes into text and identifying what those bytes were supposed to be after they were turned back.
* SMTP, the delivery mechanism responsible for shipping messages between systems, provides no quality-of-service guarantees

* Message routing is also notably absent. 
    * Email has an origin point and a destination point. It can be routed through multiple intermediaries while in transit.
    * But the sender has no way of knowing this in advance, no way to specify it, and no reasonable expectation that the intermediaries will do anything interesting with the message.


* Email is insecure. 
    * There is no native support for encryption, no support for message signing, and no mechanism for any kind of use authorization. 
    * Anyone can easily send a message to anyone, identifying themselves as anyone else. 
    * Email messages, without some extension, can be repudiated: the supposed sender can plausibly deny responsibility. 
    * It’s harder to intercept email meant for someone else, but still very possible.

### Sending Email
* Instantiate a MimeMessage object, add content, and send via a transport. 
* Adding file attachments is only a little more complex: simply create a message part for each component (body text, file attachments, etc.), add them to a multipart container, and add the container to the message.

    ```
    try {
      //...
      Session session = Session.getDefaultInstance(props, null);
      //... 
      Message msg = new MimeMessage(session);
      msg.setFrom(new InternetAddress("logs@company.com"));
      msg.setRecipient(Message.RecipientType.TO, new InternetAddress("root@company.com"));
      msg.setSubject("Today's Logs");
      //...
      Multipart mp = new MimeMultipart(  );
      MimeBodyPart mbp1 = new MimeBodyPart(  );
      mbp1.setContent("Log file for today is attached.", "text/plain");
      //...
      mp.addBodyPart(mbp1);
      //... 
      File f = new File("/var/logs/today.log");
      MimeBodyPart mbp = new MimeBodyPart(  );
      mbp.setFileName(f.getName(  ));
      mbp.setDataHandler(new DataHandler(new FileDataSource(f)));
      mp.addBodyPart(mbp);
       . . . 
      msg.setContent(mp);
      Transport.send(msg);
    } catch (MessagingException me) {
        me.printStackTrace(  );
    }
    ```

### Use Dynamic Content Strategies
### Use XML for Content
### Use Templates for Repeated Content
### Use Multipart/Related for Rich Messages

### Consider Email as an Enterprise Bridge
* When implementing systems that receive email, consider existing options before creating your own. 
* The James Mail Server, from the Apache Software Foundation, is a pure Java SMTP, IMAP, and POP mail server. 
* James is built around the JavaMail APIs and introduces the concept of a “maillet,” which is somewhat like a Java servlet but processes incoming email instead of incoming HTTP requests. 

### Retrieve Incoming Mail Efficiently
### Use Inbox Listeners Carefully

### Choose an Effective Delivery Mechanism
* IMAP mailbox
* POP3 mailbox

### Beware Email in Transactions
* The transaction support in J2EE was not intended to handle this kind of activity. 
* If an EJB session bean uses JavaMail to send email or delete a message from a mailbox, rolling the transaction back will not unsend or restore the messages.
* When sending messages, try to hold off until the very end of a transaction, and try to design your transactions so that user input, particularly if provided via email that might arrive at an indefinite time in the future, comes at a break between transactions. 
* If the transaction must send an email to a live user early in the process, it should send the user another message if the transaction is rolled back.

### Cut and Run
* The best practice for incorporating incoming email into your applications is to get the message out of the email layer as quickly as possible and into a transaction that can be controlled more tightly.
	* The mailbox should never be used as a receptacle for data related to an ongoing process. Instead, the mail should be read out as quickly as possible and loaded into another queue. 
	* A database is an obvious choice here.

* When retrieving messages, log in, do your business, and log out as quickly as possible. 
	* Staying connected and periodically polling for new messages consumes network resources, which is reason enough to try and avoid doing this, but this also makes it much more difficult to write code that will properly handle dropped connections to the mail server.

* Connecting to most well-written servers will obtain a lock on the mailbox. 
	* If multiple threads access a mailbox at the same time, each thread or application should be aware of the possibility of conflict.
	* For the same reason, applications should be sure to close connections as soon as possible to free up access to the resource.

* Information about the state of a mailbox should be considered out-of-date as soon as the connection to the mailbox is closed. 
	* This is particularly true of the message IDs provided by the message store, which are not constant or unique beyond the context of the current connection.
	
## References
* [1][Java Enterprise Best Practices - by O'Reilly Java Authors](https://www.oreilly.com/library/view/java-enterprise-best/0596003846/)