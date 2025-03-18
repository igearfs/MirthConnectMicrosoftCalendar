# Microsoft Graph API Integration with Mirth Connect (Calendar Scheduling)

## ⚠️ **DISCLAIMER – USE AT YOUR OWN RISK**
🚨 **This software is provided "as is", without warranty of any kind.**  
📌 **You assume all risks related to using this code, including but not limited to data loss, security vulnerabilities, and unexpected behavior.**  
📌 **This project does NOT guarantee HIPAA compliance or any legal protections.**  
📌 **Test thoroughly before production and modify as needed.**  

---

## 📜 **Server Side Public License**
📌 **This project is licensed under the [Server Side Public License (SSPL) v1.0](https://www.mongodb.com/licensing/server-side-public-license).**  

---

## 📌 Features
✅ **Creates, Updates, and Deletes Calendar Events** in **Microsoft Outlook**  
✅ **Finds Events using Transaction ID (`SCH-1`)** stored in `categories[]`  
✅ **Stores Configuration in the Mirth Connect Configuration Map**  
✅ **All calls should be using the prefix: https://graph.microsoft.com

---

## 🔧 **1. Mirth Connect Configuration (Using Configuration Map)**  
Store credentials securely in **Mirth Connect → Settings → Configuration Map**.

### **🔹 Add These Keys:**
| **Key** | **Value** |
|-------------|----------|
| `office_tenant_id` | **Your Azure Tenant ID** |
| `office_client_id` | **Your Microsoft Graph App Client ID** |
| `office_secret_value` | **Your Client Secret** |

---

## 📡 **2. HL7 SIU_S12 Message Example**
```hl7
MSH|^~\&|MIRTH|SCHEDULER|HOSPITAL|HIS|20250310120000||SIU^S12|EXC-43ER-32EGERG|P|2.5.1
SCH|EXC-43ER-32EGERG|||||Meeting with Team 2||20250310140000|20250310150000|60|MIN|Confirmed|||||
PID|1||123456^^^HOSPITAL^MR||Doe^John||19800101|M|||123 Main St^^New York^NY^10001||555-1234|john.doe@example.com||||
PV1|1|O|Clinic 1||||12345^Smith^Jane^^^^MD|||||||||||||||||||||||||||||||||20250310120000
RGS|1|123456
AIS|1||20250310140000|60|MIN|||Doe^John
AIG|1||12345^Smith^Jane^^^^MD||||||||
NTE|1||Doctor Email: jane.smith@hospital.com

📝 3. How We Store transactionId

We store the HL7 SCH-1 (Transaction ID) in categories[] in Microsoft Graph API.
📌 JSON Format (Microsoft Graph Event)

{
   "subject": "Meeting with Team 2",
   "start": { "dateTime": "2025-03-10T14:00:00", "timeZone": "UTC" },
   "end": { "dateTime": "2025-03-10T15:00:00", "timeZone": "UTC" },
   "attendees": [
      { "emailAddress": { "address": "someone@example.com", "name": "John Doe" }, "type": "required" }
   ],
   "body": { "content": "Initial event created via Mirth.", "contentType": "text" },
   "categories": ["TransactionId:EXC-43ER-32EGERG"]
}

🔍 4. Finding an Event by Transaction ID

We retrieve the event using categories[].
🔍 Search Request (Using Categories)
"categories": ["TransactionId:EXC-43ER-32EGERG"]  <-- so this is your lookup in a caledar for a user.

https://graph.microsoft.com/v1.0/users/{userId}/events?$filter=categories/any(a:a eq 'TransactionId:EXC-43ER-32EGERG')

🚨 5. Caveats to Watch Out For
🚨 Transaction ID Consistency Per Event

The transaction ID (SCH-1) MUST remain the same across all SIU messages for the same event for create, update, and delete operations.
If it changes, we will not find the event, unless we pass the event ID or use another way to track it.

/*** THIS NEEDS TO BE THE SAME ACROSS ALL SIU MESSAGES FOR THE SAME EVENT ***/
var messageTransactionId = "TransactionId:" + msg['SCH']['SCH.1']['SCH.1.1'].toString();

If that is in a different spot then assign it to yours.

🚨 HL7 Fields Used for Scheduling

/*** OTHER INFO ***/
var patientEmail = msg['PID']['PID.14']['PID.14.1'].toString();
var patientName = msg['PID']['PID.5']['PID.5.2'].toString() + " " + msg['PID']['PID.5']['PID.5.1'].toString();
var calendarSubject = msg['SCH']['SCH.6']['SCH.6.1'].toString();

/*** YES I AM CHEATING AND HARDCODING THESE VALUES ****/
var startDate = "2025-03-10T14:00:00";
var endDate = "2025-03-10T15:00:00";
var visitReason = "Some information here";

/**** CREATE EVENT JSON PAYLOAD ****/
var event = {
   subject: calendarSubject,
   start: { dateTime: startDate, timeZone: "UTC" },
   end: { dateTime: endDate, timeZone: "UTC" },
   attendees: [{ emailAddress: { address: patientEmail, name: patientName }, type: "required" }],
   body: { content: visitReason, contentType: "text" },
   categories: [
     messageTransactionId
  ]
};

🚨 Storing Provider Outlook Email in Mirth

📌 The system must retrieve the doctor email from their own tenant.
📌 This implementation does not assume a specific HL7 field for provider email.
📌 Modify accordingly based on how the provider email is supplied.
📜 6. Licensing

📌 This project is licensed under the [Server Side Public License (SSPL) v1.0](https://www.mongodb.com/licensing/server-side-public-license).

📞 7. Support

For issues, open a GitHub Issue.
