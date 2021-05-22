```python
"""
Process emails deposited into S3 by SES and send them
all to my gmail address.

References:
https://erlerobotics.gitbooks.io/erle-robotics-python-gitbook-free/e-mail_composition_and_decoding/understanding_mime.html
"""

import smtplib
import os

import boto3
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
import email.utils
import urllib.parse


s3 = boto3.client('s3')
ses = boto3.client('ses')


def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(
        event['Records'][0]['s3']['object']['key'],
        encoding='utf-8'
    )

    # Get the object
    message_object = s3.get_object(Bucket=bucket, Key=key)

    # Make an `email` object from the... object
    email_object = email.message_from_bytes(message_object['Body'].read())

    subject = email_object.get('Subject')
    original_sender = email_object.get('From')
    sender = os.getenv('SENDER')
    recipient = os.getenv('RECIPIENT')

    smtp_username = os.getenv('SMTP_USERNAME')
    smtp_password = os.getenv('SMTP_PASSWORD')
    region = os.getenv('REGION', 'us-east-1')

    # Create message container - the correct MIME type is multipart/alternative
    message = MIMEMultipart('alternative')
    message['Subject'] = subject
    message['From'] = email.utils.formataddr((original_sender, sender))
    message['To'] = recipient
    message.add_header('reply-to', original_sender)

    # Check for text/plain
    text_part = [
        _ for _ in email_object.walk()
        if _.get_content_type() == 'text/plain'
    ]
    text_payload = '' if not text_part else text_part[0].get_payload(decode=True).decode('utf-8')

    # Check for text/html
    html_part = [
        _ for _ in email_object.walk()
        if _.get_content_type() == 'text/html'
    ]
    html_payload = None if not html_part else html_part[0].get_payload(decode=True).decode('utf-8')

    # Make a list of attachments
    attachment_parts = [
        _ for _ in email_object.walk()
        if _.get_content_disposition() in ['attachment', 'inline']
    ]

    # Attach parts into message container.
    # According to RFC 2046, the last part of a multipart message, in this case
    # the HTML message, is best and preferred.
    message.attach(MIMEText(text_payload, 'plain'))
    if html_payload:
        message.attach(MIMEText(html_payload, 'html'))

    for attachment in attachment_parts:
        _ = MIMEBase(
            attachment.get_content_maintype(),
            attachment.get_content_subtype(),
        )

        _.add_header(
            'Content-Transfer-Encoding',
            attachment['Content-Transfer-Encoding'],
        )
        _.add_header(
            'Content-Disposition',
            'attachment',
            filename=attachment.get_filename(),
        )
        _.set_payload(
            attachment.as_bytes(),
            charset=attachment.get_charsets()[0],
        )

        message.attach(_)

    # Try to send the message.
    try:
        server = smtplib.SMTP(
            host=f'email-smtp.{region}.amazonaws.com',
            port=587,
        )

        server.ehlo()
        server.starttls()
        server.ehlo()
        server.login(smtp_username, smtp_password)
        server.sendmail(sender, recipient, message.as_string())
        server.close()

    # Display an error message if something goes wrong.
    except Exception as e:
        print('Error: ', str(e))
```
