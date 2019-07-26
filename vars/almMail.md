# [almMail](/vars/almMail.groovy)

Sends a notification mail with build result. Usually, this step is called in a pipeline post step.  

**Parameters:**
- *recipientProviders:* if given email recipients (by default RequesterRecipientProvider, DevelopersRecipientProvider and CulpritsRecipientProvider)
- *subject:* if given email subject (by default $DEFAULT_SUBJECT)
- *content:* if given email content (by default $JELLY_SCRIPT)
- *replyTo:* if given reply to these recipients (by default $DEFAULT_REPLYTO)
- *attachLog:* if true log is attached (by default only in failed builds)
- *keepNullResult* if true currentBuild.result == null is not considered as 'SUCCESS' so the build is considered to still running (false by default).
- *to:* if given email is also sent to these emails (separated by ';')
- *mimetype:* Mimetype (html/text by default)

```groovy
post {
  changed {
    almMail()
  }
}
```