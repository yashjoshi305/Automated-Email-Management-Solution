# Automated Email Management Solution - Process Explanation

## Overview

This solution automates the management of emails received in a shared Outlook mailbox. It's designed to fetch new emails, analyze their sentiment and classify the type of request, and then automatically forward them to the appropriate person or team. The system also logs the process and uploads data to Google BigQuery for historical tracking and analysis. This document outlines the step-by-step process implemented by the solution.


## Business Problem Addressed

* **Clients respond to emails in a shared mailbox (Outlook).**
    * **Solution:** The intended functionality of `1_fetch_clean.ipynb` is to connect to the shared Outlook mailbox using IMAP, directly addressing the source of the client requests.

* **Manually monitored and routed based on received date (LIFO), escalation (sentiment analysis), and priority markers.**
    * **Solution:**
        * **Received Date:** While not the primary routing factor in the current `3_forward.ipynb`, the received date would be extracted by `1_fetch_clean.ipynb` and stored in BigQuery by `4_upload.ipynb`, enabling potential future analysis or routing rules based on this.
        * **Escalation (Sentiment Analysis):** `2_analyse.ipynb` performs sentiment analysis, and the resulting sentiment is included in the forwarded email body in `3_forward.ipynb`, flagging potential escalations.
        * **Priority Markers:** The current solution does not explicitly handle priority markers. Implementation in `1_fetch_clean.ipynb` to identify and `3_forward.ipynb` to act upon these markers would be a future enhancement.

* **Routing based on type of work, sender etc.**
    * **Solution:**
        * **Type of Work:** `2_analyse.ipynb` classifies emails by type, and `3_forward.ipynb` routes them based on this classification using a predefined mapping.
        * **Sender:** While the sender's information is captured and stored, the current routing logic in `3_forward.ipynb` does not utilize it. Future enhancements could include routing rules based on the sender.

* **Manual process needing upliftment and automation.**
    * **Solution:** The entire architecture, with scheduled scripts for fetching, analyzing, and forwarding, automates the manual process, providing upliftment through intelligent classification and routing.

## Expected Outcome Achieved

* **Emails are automatically read based on a schedule.**
    * **Solution:** `0_run.ipynb` schedules `1_fetch_clean.ipynb` to run daily, automating the reading of new emails from the shared mailbox (upon full implementation of fetching logic).

* **Segregated on sentiment analysis and classified on type of requests.**
    * **Solution:** `2_analyse.ipynb` performs both sentiment analysis and request classification, adding this information to the email data.

* **These emails are then automatically routed to the person available to be worked.**
    * **Solution:** `3_forward.ipynb` automatically routes emails based on the classified type. Availability checks are not currently implemented but could be a future integration.

* **Quality assured to ensure no misses and appropriate logging for debugging.**
    * **Solution:** Scheduled execution aims to prevent misses, though explicit tracking of processed emails could be improved. `0_run.ipynb` and individual scripts provide basic logging for debugging.

* **Stores historical data.**
    * **Solution:** `4_upload.ipynb` stores both raw and analyzed email data in Google BigQuery for historical tracking.

* **MI & Analytics to analyze trends on process gaps and recommendations.**
    * **Solution:** The data in BigQuery (from `4_upload.ipynb`) provides the foundation for MI and analytics, enabling the identification of trends and process gaps, although the analysis and visualization steps are not explicitly included in the provided code.

## E2E Working Solution Design Implementation

* **Data extraction and clean up:** The intended functionality of `1_fetch_clean.ipynb` aims to extract and perform initial cleanup of email data.
* **Storage of data into a DB:** `4_upload.ipynb` implements the storage of data into Google BigQuery.
* **Data wrangling and cleaning:** `2_analyse.ipynb` performs data wrangling to prepare the data for analysis. More comprehensive cleaning would ideally be in `1_fetch_clean.ipynb`.
* **Data analysis and visualization:** `2_analyse.ipynb` performs the core analysis. The stored data in BigQuery is ready for visualization using BI tools or further scripts.
* 
## Solution Process

The automated email management process is broken down into four main stages, executed sequentially based on a defined schedule:

**1. Email Fetching and Initial Data Handling (`1_fetch_clean.ipynb` - Scheduled to run daily at 1000 hrs every day):**

* **Intended Functionality (Currently Lacking Implementation in Provided Code):** This stage is intended to connect to a configured shared Outlook mailbox. It would identify and retrieve new, unread emails.
* **Data Extraction:** Once emails are fetched, the script would extract key information from each email, such as:
    * Sender's email address
    * Recipient's email address (the shared mailbox)
    * Subject line
    * Email body/content
    * Unique Identifier (UID) of the email
    * Received date and time
* **Initial Data Storage:** The extracted information would then be organized into a structured format, likely using the pandas library to create a DataFrame.
* **Saving Initial Data:** The DataFrame containing the raw email data is saved to a local pickle file named `emails.pkl`. Additionally, a timestamped version of this file (e.g., `emails_20250410_210900.pkl`) is also saved for potential historical snapshots.

**2. Email Analysis (`2_analyse.ipynb` - Scheduled to run daily at 1015 hrs):**

* **Loading Initial Data:** This stage begins by loading the DataFrame containing the email content from the `emails.pkl` file created in the previous step.
* **Request Type Classification:** For each email's content, a zero-shot classification model (`MoritzLaurer/DeBERTa-v3-base-mnli-fever-anli`) is used to categorize the type of request. It attempts to classify the email into predefined categories such as "technology," "News," "Job search," etc. The predicted type and its confidence score are recorded.
* **Urgency Assessment:** Another zero-shot classification model (`facebook/bart-large-mnli`) is employed to determine the urgency of the email. It classifies the email into categories like "blocker," "major," "normal," or "minor," along with a confidence score.
* **Sentiment Analysis:** The email content is also analyzed for sentiment using a pre-trained sentiment analysis model (`cardiffnlp/twitter-roberta-base-sentiment-latest`). The model predicts whether the sentiment is positive, negative, or neutral, and provides a confidence score.
* **Storing Analysis Results:** The results of the type classification, urgency assessment, and sentiment analysis (the predicted labels and their confidence scores) are added as new columns to the pandas DataFrame. A subset of the analyzed DataFrame (currently the first two rows in the provided code) is then saved to a pickle file named `type.pkl`.

**3. Email Forwarding (`3_forward.ipynb` - Scheduled to run daily at 1030 hrs):**

* **Loading Analysis Results:** This stage loads the analyzed email data from the `type.pkl` file. This DataFrame is expected to contain the original email's UID, the classified type, urgency, and sentiment.
* **Recipient Mapping:** A predefined Python dictionary (`emails`) maps the classified email type to a specific recipient email address. For example, emails classified as "technology" are directed to `yashjoshi30052@gmail.com`.
* **Email Retrieval and Forwarding:** For each email in the DataFrame:
    * The script connects to a configured Gmail account using the IMAP protocol to access the original email in the shared mailbox (assuming the UID is available).
    * It fetches the full content of the original email using its UID.
    * A new email message is created with the following structure:
        * **Subject:** "Fwd: " followed by the original email's subject.
        * **From:** The configured forwarding Gmail account.
        * **To:** The recipient email address determined by the email type.
        * **Body:** Includes a header indicating it's a forwarded message, the original sender and subject, the determined urgency and sentiment, followed by the original email's body.
    * The newly created forward email is then sent using the SMTP protocol via the configured Gmail account.
    * The script prints a confirmation message for each successfully forwarded email.

**4. Data Upload to Google BigQuery (`4_upload.ipynb` - Scheduled to run daily at 1045 rs):**

* **Loading DataFrames:** This stage loads two pandas DataFrames: one from `emails.pkl` (the initial fetched data) and another from `type.pkl` (the analyzed data).
* **Connecting to BigQuery:** The script uses the Google Cloud BigQuery client, authenticated using a Service Account key file (`client_secret.json` and `fetchemail-3005-822740b3bf5b.json`). This allows the script to interact with Google BigQuery.
* **Data Ingestion:** The two DataFrames are then uploaded to Google BigQuery tables within the specified project (`fetchemail-3005`) and dataset (`Logs`).
    * The data from `emails.pkl` is appended to a table named `email_fetch_<timestamp>`, where `<timestamp>` represents the date and time of the upload.
    * The data from `type.pkl` is appended to a table named `email_type_<timestamp>`.
* **Confirmation:** The script prints a message confirming the successful upload of the DataFrames to the respective BigQuery tables.

**Scheduling and Execution (`0_run.ipynb`):**

* This script utilizes the `schedule` library to automate the execution of the other three scripts (`1_fetch_clean.ipynb`, `2_analyse.ipynb`, `3_forward.ipynb`, and `4_upload.ipynb`) at specific times each day.
* It defines the schedule for each script 
* A continuous `while True` loop ensures that the script keeps checking for any pending scheduled tasks and executes them when their scheduled time arrives.
* The `subprocess` module is used to run each of the other `.ipynb` files as separate Python processes.

This detailed explanation outlines the intended workflow and the logic within each of the provided code files, offering a clear understanding of how the automated email management solution is designed to operate.
