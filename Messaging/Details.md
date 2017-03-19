
The Messaging plugin makes it possible to make a phone call, send a sms or send an e-mail using the default messaging applications on the different mobile platforms.

### API Usage

The Messaging Plugin makes use of `IEmailTask`, `ISmsTask` and `IPhoneCallTask` abstractions to send an e-mail, send a sms or make a phone call respectively.  These abstractions are defined within the `Plugin.Messaging.Abstractions` PCL library.  Platform specific implementations for these different abstractions are provided within a `Plugin.Messaging` library for the different platforms.

```csharp
public interface IEmailTask
{
    bool CanSendEmail { get; }
    bool CanSendEmailAttachments { get; }
    bool CanSendEmailBodyAsHtml { get; }
    void SendEmail(IEmailMessage email);
    void SendEmail(string to, string subject, string message);
}
```

```csharp
public interface ISmsTask
{
    bool CanSendSms { get; }
    void SendSms(string recipient, string message);
}
```

```csharp
public interface IPhoneCallTask
{
	bool CanMakePhoneCall { get; }
    void MakePhoneCall(string number, string name = null);
}
```

### Using the API 
The messaging API's can be accessed on the different mobile platforms using the `CrossMessaging` singleton or `MessagingPlugin` static class.  Have a look at the `Plugin.Messaging.Samples.sln` for samples that illustrate using the API on the different platforms. Here are some snippets from the samples that illustrate how to access the API from within an `Activity`, `UIViewController` or Windows `Page`.  

```csharp
// Make Phone Call
var phoneDialer = CrossMessaging.Current.PhoneDialer;
if (phoneDialer.CanMakePhoneCall) 
	phoneDialer.MakePhoneCall("+27219333000");

// Send Sms
var smsMessenger = CrossMessaging.Current.SmsMessenger;
if (smsMessenger.CanSendSms)
   smsMessenger.SendSms("+27213894839493", "Well hello there from Xam.Messaging.Plugin");

var emailMessenger = CrossMessaging.Current.EmailMessenger;
if (emailMessenger.CanSendEmail)
{
	// Send simple e-mail to single receiver without attachments, bcc, cc etc.
	emailMessenger.SendEmail("to.plugins@xamarin.com", "Xamarin Messaging Plugin", "Well hello there from Xam.Messaging.Plugin");

	// Alternatively use EmailBuilder fluent interface to construct more complex e-mail with multiple recipients, bcc, attachments etc. 
    var email = new EmailMessageBuilder()
      .To("to.plugins@xamarin.com")
      .Cc("cc.plugins@xamarin.com")
      .Bcc(new[] { "bcc1.plugins@xamarin.com", "bcc2.plugins@xamarin.com" })
      .Subject("Xamarin Messaging Plugin")
      .Body("Well hello there from Xam.Messaging.Plugin")
      .Build();

    emailMessenger.SendEmail(email);
}           
```

### Platform specific API's

Sending HTML e-mail and adding e-mail attachments are only supported on some platforms.  Use the ```IEmailTask.CanSendEmailAttachments``` and ```IEmailTask.CanSendEmailBodyAsHtml``` API's to test whether the feature is available for the platform in your PCL code.  

To add HTML body content use ```EmailMessageBuilder.BodyAsHtml``` (**iOS, Android**).  

```csharp
// Construct HTML email (iOS and Android only)
var email = new EmailMessageBuilder()
  .To("to.plugins@xamarin.com")
  .Subject("Xamarin Messaging Plugin")
  .BodyAsHtml("Well hello there from <a>Xam.Messaging.Plugin</a>")
  .Build();
```

To add attachments, use the ```EmailMessageBuilder.WithAttachment``` overloads.  There are platform specific overloads that will allow you to attach a `Windows.Storage.IStorageFile` (**UWP**), `Java.IO.File` (**Android**) and `Foundation.NSUrl` (**iOS**).  Alternatively use the `WithAttachment(string, string)` overload to attach a file from within a PCL project. **Please note that on the Windows platform, attaching from the PCL only works for files contained within the ApplicationData due to the security restrictions of the platform**.  

```csharp
// Construct email with attachment from a PCL
var email = new EmailMessageBuilder()
  .To("to.plugins@xamarin.com")
  .Subject("Xamarin Messaging Plugin")
  .Body("Well hello there from Xam.Messaging.Plugin")
  .WithAttachment("/storage/emulated/0/Android/data/MyApp/files/Pictures/temp/IMG_20141224_114954.jpg", "image/jpeg");
  .Build();
```