---
layout: post
title:  "Sending and Receiving SMS with the Twilio API"
date:   2016-04-14 13:22:17
categories:
---

You’ve built an application that needs to send and receive text messages. Lucky for you, there is an API for that! In this short tutorial, you will learn how to connect your Rails application to the Twilio API.

## First, set up a Twilio account

- Go to [Twilio](http://www.twilio.com)’s website
- Make an account and add a phone number

I chose a number with my local area code, and named it after my application on the ‘Phone Numbers’ tab of Twilio’s site. You will need the number to have SMS enabled, so be sure the number has SMS listed as a ‘capability’ on the detail view under the Phone Numbers tab on the Twilio site.

You now have a trial account, and you can confirm only one personal number for your application to communicate with before being required to purchase anything. For the purpose of testing your application, add and follow the instructions to confirm your own cell phone number. 

- After confirming your phone, go to the Dev Tools tab on the Twilio site, then click on the ‘API Explorer’ tab beneath that.



![API Explorer](/assets/Dev-tools-api-explorer.png)



- You should see a Send Message page, where you can enter a ‘To’ number under ‘Parameters’.



![Send Message Form](/assets/Explorer-send-message.png)



- Enter your number, and type a message into the ‘Body’ textbox. You can also insert an image URL into the appropriate box if you like. 
- Go to the bottom of the page, taking note for future reference of the copyable code in the programming language you select. 
Example:



![API Explorer](/assets/Request-sample-code.png)



An example of the Ruby code looks something like:

```ruby

   require 'rubygems' 
```

*not necessary with ruby 1.9 but included for completeness* 

   require 'twilio-ruby' 
 
*Put your own credentials here:*

account_sid = 'AC7f31bedb178e5c82381712d702187f19' 
auth_token = '[AuthToken]' 
 
## Set up a client to talk to the Twilio REST API 

@client = Twilio::REST::Client.new account_sid, auth_token 
 
@client.account.messages.create({
  :from => '+14025551212',
  })

   ```
- Click the ‘Make Request’ button. You will only be able to text to the number that you confirmed earlier.

Take a sip of coffee, and you will receive a text message! 

- Now, go to your ‘Account Settings’ and find your API Credentials. 
- You will see two sets of credentials: ‘Live’ and ‘Test’. The ‘Test’ credentials will only emulate the sending of SMS, and so we will use the ‘Live’ credentials for this example.
- To get your site up and running, copy your **AccountSID** _and_ your **AuthToken**.
- Go to your Rails project in your text editor and paste these codes into an `.env.local` file that has been listed in a `.gitignore` file in the same directory. In your `.env` file that is synced with GitHub, enter ‘foo’ as the values. They will be placeholders for the time being. 

- Example, in `.env` file:


   ```ruby
   TWILIO_ACCOUNT_ID=foo
   TWILIO_AUTH_TOKEN=foo
   ```

- And in your `.env.local` file, put your real credentials. These will be ignored by Git, and so will not be visible to anyone else publicly:

   ```ruby
   TWILIO_ACCOUNT_ID='AC7f31bedb178#####################'
   TWILIO_AUTH_TOKEN='0b67f2ae87cae#####################'
   ```

- Next we will need to install the twilio-ruby gem. Go to your Gemfile.rb, and add `gem "twilio-ruby"` to your list. 
- Run `bundle install` in Terminal to reload your gems and give Twilio access to your application!

- Open your `routes.rb` file in the editor. Create a route that calls the controller action defined in your `controller.rb` file that will send the text message.

##For Example:##

 In `routes.rb`:

  >`post "dashboard/business/send_message/:client_id"  => "touch#send_sms"`

This points to the `send_sms` action in **touch_controller.rb**.



Remember the code snippet at the bottom of the API Explorer page on the Twilio site?
Go there and click on the ‘Ruby’ tab (or whichever language you need, of course) to get the code and copy it to your clipboard.


In your controller file, reference your API credentials in the ‘env’ file:

    ```ruby
    account_sid = ENV["TWILIO_ACCOUNT_ID"]
    auth_token = ENV["TWILIO_AUTH_TOKEN"]``` 

 ##Setting up a ‘client’ to make a request to Twilio:##
(note: we are speaking here in the server/client sense of the word- this is different from our Client object) 

Essentially, the client we create below is the object that has the quality of being able to communicate with our Twilio API:

    ```ruby
    @client = Twilio::REST::Client.new(account_sid, auth_token)
   ``` 
Note: The Read Me at the <a href=”https://github.com/twilio/twilio-ruby”>twilio-ruby</a> repository may help you here if this bit sounds confusing.     

 ##Sending an SMS, complete with an image, using Twilio’s format for setting the parameters:##

   ```ruby 
   @client.account.messages.create(
      :from => text_sender_business.business_phone, 
      :to => text_recipient.phone_number, 
     :body => text_content 
      :media_url => 'http://farm2.static.flickr.com/1075/1404618563_3ed9a44a3a.jpg'
   )
   redirect_to "/dashboard/business/current_thread/#{text_recipient.id}"
   end
   ```

Now, to use SMS to `receive` messages to our application, we must have a pathway from Twilio to our application. Note that your local server will not work for this, as our site is not yet deployed. If you are in a similar situation, and would like to troubleshoot all of this prior to deploying your site, you may choose to use something like Ngrok. Ngrok will essentially “listen” to a port of your choosing (the port on which you are running your application locally), and put it on Ngrok’s server under a temporary url. This means that your application will be able to communicate with Twilio during this time.  

- Go to:<a href=”https://ngrok.com”>Ngrok</a> and download the application. Place the application in a root directory, and install with Terminal.

##To set ngrok to listen to localhost port 3000:##

- In command line, from the directory where the ngrok application is, run `./ngrok http 3000`

When you start the server, you will see  something like this:

```
Tunnel Status                 online
Version                       2.0.25/2.0.25
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://d94790ff.ngrok.io -> localhost:3000
Forwarding                    https://d94790ff.ngrok.io -> localhost:300

Connections                   ttl     opn     rt1     rt5     p50     p9
                            0       0       0.00    0.00    0.00    0.
```

- Using the example above, take <http://d94790ff.ngrok.io>, and paste it into your Twilio account’s ‘Request URL’ textbox on your phone number’s ‘Configure’ page.

<img src=”https://goo.gl/photos/VFEXMNY2F7STmZPc9”>

>##Important!##
>The above tunnel to your ngrok site will change each time you start your ngrok listener. You will need to go to Twilio’s config page again and paste in the new location from the running listener’s status in Terminal.  

Next, you will need to give ngrok something to listen to. Start your Rails server in a separate tab in Terminal:

`rails s`

Ngrok is now listening to your local Rails server and making it available on the web. Test this by going to Ngrok using the url from above: My example: <http://d94790ff.ngrok.io”>. You should see your website!

Next, if you are using an authentication method for your site, you will need to make sure Twilio has a path around that. My application is using Devise (a Ruby gem), and so I have put   `skip_before_filter :authenticate_user!, only: [:save_incoming_sms]` 
at the top of my `touch_controller.rb` file.

##For incoming text messages:##

In `routes.rb`:

  ```ruby
get "touch/incoming" => "touch#save_incoming_sms"``` 


And in ‘touch_controller.rb`:
```ruby
    def save_incoming_sms
    t = Touch.new
    x = Client.where("phone_number" => params[:From].last(10))

    x.each do |e|
      t.client_id = e.id
    end
    
    t.message = params[:Body]
    t.outgoing = false
    t.read = false
    t.save

    render nothing: true

   end
```

##A Brief Explanation of the Code Above:##

When Twilio receives an SMS to the number you chose in your account (assuming you are running your Rails server and the Ngrok listener), Twilio references the url of the Ngrok site (the temporary url you gave to Twilio, and sends a request which hits the “touch/incoming” path in the `routes.rb` file. 

That calls the controller action `save_incoming_sms` on our `touch` object. The message sent from Twilio has more information than I used here, so inspect it in your Rails console if you are curious. My app looks at the params `:From` in the message object, and searches for a user of my site with that phone number.

Once the appropriate user is found, their user_id is entered in the corresponding column for this message instance, and the body is saved. I also set two boolean values- one for `read` and one for `outgoing`. These values will help when it comes to displaying the message on my site. 
 
To be able to see your message and to test that our API is operating and its values being saved in the proper place (without yet writing the code needed to query the database and display it), I use the database viewer <a href= “https://eggerapps.at/postico/”>Postico</a>. Now when you send an SMS from your application, it will appear on your phone and when you reply it will appear in your database! Remember to refresh Postico to see your message appear. It is now a matter of picking it up from there to use in your views! 

Remember: the Ngrok url must be noted in the `Forwarding` line in Terminal `each time` Ngrok is run, and then entered (where you entered it initially) on the `request url` text box in the Manage tab below Phone Numbers on the Twilio site.

Thank you for taking the time to read this- I do hope you found some value in it. Feedback is always appreciated, so please do not hesitate to contact me with complaints and/or suggestions.





 


