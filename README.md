# n8n-omada-tp-link-monitor-pushover
This repository contains a set of n8n workflows for automating TP Link Omada network monitoring. It processes email alerts from your controller, logs device status to a Google Sheet, and sends high-priority Pushover alerts for prolonged device disconnections. This is a complete, open-source solution for keeping tabs on your network's health.


# Omada Network Monitoring Workflow

This n8n workflow is designed to monitor device connection status on an Omada network. It processes alerts sent from the Omada controller via email, logs the status changes to a Google Sheet, and sends immediate push notifications for devices that have been disconnected for an extended period.

## Workflow Overview

The workflow is broken into three main parts:

1.  **Email Processing (`Receives Alert` -> `Append Row in Sheet`)**: This part of the workflow is triggered whenever a new email is received. It uses a Code node to parse the raw text of the email, extracting details like the device name, MAC address, and connection status. This data is then formatted and appended as a new row to a Google Sheet, creating a comprehensive log of all connection events.

2.  **Disconnected Device Alerting (`Check Every 5 minutes` -> `Alert User 1` & `Alert User 2`)**: This is a scheduled part of the workflow that runs every 5 minutes. It reads the entire device log from the Google Sheet, identifies the most recent status for each unique device, and then filters for any devices that are currently disconnected. If a device has been disconnected for more than 30 minutes, it sends a high-priority push notification to two different Pushover accounts. Once an alert is sent, it updates the corresponding row in the Google Sheet to prevent duplicate alerts.

3.  **Data Maintenance (`Clear Rows Every 2 days` -> `Clear sheet`)**: A separate scheduled trigger runs every two days to perform cleanup on the Google Sheet. This helps to keep your log from getting too large by deleting the oldest rows.

## Services and Dependencies

To get this workflow running, you'll need the following services connected to your n8n instance:

* **Google Sheets**: Used as the primary database to log all connection events and track the status of devices.
* **Gmail**: Acts as the trigger for the initial workflow, listening for alerts from your Omada controller.
* **Pushover**: Used to send real-time, high-priority push notifications to mobile devices.

## Setup Instructions

1.  **Create a Google Sheet Log**:
    * Create a new Google Sheet in your Google Drive.
    * Name the sheet something descriptive like "Omada Device Log".
    * In the first row, create the following headers exactly as they are listed: `rowId`, `timestamp`, `timestampISO`, `category`, `severity`, `mac`, `name`, `type`, `status`, `checkAfter`, `alertSent`, and `timeStampFormated`. This is crucial for the workflow to correctly map the data.

2.  **Configure Credentials**:
    * **Google Sheets**: In n8n, create a new Google Sheets OAuth2 API credential.
    * **Gmail**: Create a new Gmail OAuth2 credential. This email account must be set up to receive the alerts from your Omada controller.
    * **Pushover**: Create a new Pushover API credential. You will also need to find the user keys for each person you want to notify.

3.  **Import the Workflow**:
    * Copy the cleaned JSON workflow code.
    * In your n8n instance, click "New" -> "Import from JSON".
    * Paste the code and import the workflow.

4.  **Update the Nodes with Your Information**:
    * **Receives Alert**: Connect this node to the Gmail credential you created.
    * **Append Row in Sheet**, **Get Row(s) in Sheet**, **Update Alert**, and **Clear sheet**: For each of these Google Sheets nodes, replace `YOUR_GOOGLE_SHEET_ID` in the parameters with the actual ID of the Google Sheet you created. You can find this ID in the URL of your spreadsheet. It's the string of letters and numbers between `/d/` and `/edit`.
    * **Alert User 1** and **Alert User 2**: In the parameters for these Pushover nodes, replace `YOUR_PUSHOVER_USER_KEY_1` and `YOUR_PUSHOVER_USER_KEY_2` with the specific user keys for the people you want to notify.

## Node Breakdown

* `**Receives Alert**` (Gmail Trigger): The starting point of the first workflow, listening for new emails.
* `**Process Email and Extract**` (Code): This node contains the JavaScript logic to parse the raw email text and extract structured data. It creates a `timestampISO` and a `checkAfter` timestamp, which is 30 minutes after the initial alert time.
* `**Append Row in Sheet**` (Google Sheets): Takes the structured data from the previous node and adds a new row to your Google Sheet log.
* `**Check Every 5 minutes**` (Schedule Trigger): The starting point for the second workflow, set to run at regular intervals.
* `**Get Row(s) in Sheet**` (Google Sheets): Reads all the data from your Google Sheet log to find the most recent status for each device.
* `**Check Device and Notify**` (Code): This node contains the logic to group the rows by MAC address and determine the latest status. It then filters for devices that are `disconnected` and have passed their 30-minute `checkAfter` time.
* `**Update Alert**` (Google Sheets): If a disconnection alert is sent, this node finds the corresponding row in the Google Sheet using the `rowId` and updates the `alertSent` column to `true`, ensuring the alert is not sent again.
* `**Alert User 1**` & `**Alert User 2**` (Pushover): These nodes take the filtered device information and send a formatted push notification. The message will include the device name and its disconnected status.
* `**Clear Rows Every 2 days**` (Schedule Trigger): The starting point for the third workflow, set to run periodically to maintain the sheet's size.
* `**Clear sheet**` (Google Sheets): This node clears the specified number of rows from the sheet, starting from row 2 to preserve the header.

## Contributing

If you find a bug or have an idea for an improvement, feel free to open an issue or submit a pull request!
