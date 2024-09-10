# Create Jenkins Alerts for Disk Space 


- create a freestyle job
- make it to run it on everyday at 9pm

```
#!/bin/bash
THRESHOLD=80
DISK_USAGE=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')
TO="1@outlook.com,2@outlook.com"
CC="1@outlook.com,2@outlook.com"
SUBJECT="Jenkins Agent Disk Space Alert!"

if [ "$DISK_USAGE" -gt "$THRESHOLD" ]; then
  MESSAGE="<html><body><h1 style='color:red;'><b>$SUBJECT</b></h1><p><b>Current Disk Space Usage of Jenkins Agent is </b> <span style='color:red;'>$DISK_USAGE%</span> <b>which is more than the threshold</b> <span style='color:red;'>$THRESHOLD%</span>. <b>Take Immediate Action!!!</b></p></body></html>"
else
  MESSAGE="<html><body><h1 style='color:green;'><b>$SUBJECT</b></h1><p><b>Current Jenkins Agent Disk usage is</b> <span style='color:green;'>$DISK_USAGE%</span>.</p></body></html>"
fi

#Send the email
{
  echo "To: $TO"
  echo "Cc: $CC"
  echo "Subject: $SUBJECT"
  echo "Content-Type: text/html"
  echo
  echo "$MESSAGE"
} | sendmail -t

```

<img width="527" alt="Screenshot 2024-09-10 at 5 59 30 PM" src="https://github.com/user-attachments/assets/6b2d23d9-069d-4b35-b088-094f654ca668">


<img width="868" alt="Screenshot 2024-09-10 at 6 02 09 PM" src="https://github.com/user-attachments/assets/61700361-5baf-4dfe-8294-9b38399aa17f">

