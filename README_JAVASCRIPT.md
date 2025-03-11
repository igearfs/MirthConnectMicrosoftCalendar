# **Microsoft Graph API - JavaScript Functions for Mirth Connect**

## ⚠️ **DISCLAIMER – USE AT YOUR OWN RISK**
🚨 **This software is provided "as is", without warranty of any kind.**  
📌 **You assume all risks related to using this code, including but not limited to data loss, security vulnerabilities, and unexpected behavior.**  
📌 **This project does NOT guarantee HIPAA compliance or any legal protections.**  
📌 **Test thoroughly before production and modify as needed.**  

---

## 📜 **Server Side Public License**
📌This project is licensed under the [Server Side Public License (SSPL) v1.0](https://www.mongodb.com/licensing/server-side-public-license).


---

## 📌 **1. JavaScript Functions Overview**
This repository contains JavaScript functions designed for **Mirth Connect** to integrate with **Microsoft Graph API**.  
Functions handle:
✅ **OAuth2 authentication**  
✅ **Calendar event creation**  
✅ **Finding events by transaction ID**  
✅ **Updating and deleting events**  

---

## 🔑 **2. Function: Get Microsoft User ID**
📌 **Purpose:** Retrieves a **Microsoft Outlook user ID** by email address.  

### **📌 Function Definition**
```javascript
/**
 * Retrieves Microsoft Outlook User ID by email.
 *
 * @param {String} accessToken - OAuth2 access token
 * @param {String} userEmail - User email address
 * @return {String} User ID if found, otherwise null
 */
function getMicrosoftUserId(accessToken, userEmail) {
    var URLEncoder = Packages.java.net.URLEncoder;
    var charset = "UTF-8";

    var encodedFilter = URLEncoder.encode("mail eq '" + userEmail + "'", charset);
    var userLookupUrl = "https://graph.microsoft.com/v1.0/users?$filter=" + encodedFilter;

    var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
    var HttpGet = Packages.org.apache.http.client.methods.HttpGet;
    var EntityUtils = Packages.org.apache.http.util.EntityUtils;

    try {
        var httpClient = HttpClients.createDefault();
        var httpGet = new HttpGet(userLookupUrl);
        httpGet.setHeader("Authorization", "Bearer " + accessToken);
        httpGet.setHeader("Content-Type", "application/json");

        var response = httpClient.execute(httpGet);
        var responseString = EntityUtils.toString(response.getEntity(), "UTF-8");
        var responseJson = JSON.parse(responseString);

        if (responseJson.value && responseJson.value.length > 0) {
            var userId = responseJson.value[0].id;
            return userId;
        } else {
            logger.error("❌ User not found in directory: " + responseString);
            return null;
        }
    } catch (e) {
        logger.error("❌ Exception while getting User ID: " + e);
        return null;
    }
}

📅 3. Function: Create Calendar Event

📌 Purpose: Creates a new event in Microsoft Outlook Calendar.
📌 Stores the HL7 SCH-1 (Transaction ID) in categories[] for future retrieval.
📌 Function Definition

/**
 * Creates an event in Microsoft Outlook Calendar using Graph API.
 *
 * @param {String} accessToken - OAuth2 access token
 * @param {String} userId - Microsoft Outlook user ID
 * @param {Object} eventData - JSON object containing event details
 * @return {String} Event ID if created, otherwise null
 */
function createEvent(accessToken, userId, eventData) {
    var calendarUrl = "https://graph.microsoft.com/v1.0/users/" + encodeURIComponent(userId) + "/events";

    var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
    var HttpPost = Packages.org.apache.http.client.methods.HttpPost;
    var StringEntity = Packages.org.apache.http.entity.StringEntity;
    var EntityUtils = Packages.org.apache.http.util.EntityUtils;

    try {
        var httpClient = HttpClients.createDefault();
        var httpPost = new HttpPost(calendarUrl);
        httpPost.setHeader("Authorization", "Bearer " + accessToken);
        httpPost.setHeader("Content-Type", "application/json");
        httpPost.setEntity(new StringEntity(JSON.stringify(eventData), "UTF-8"));

        var response = httpClient.execute(httpPost);
        var responseString = EntityUtils.toString(response.getEntity(), "UTF-8");
        var responseJson = JSON.parse(responseString);

        if (responseJson.id) {
            return responseJson.id;
        } else {
            logger.error("❌ Event creation failed: " + responseString);
            return null;
        }
    } catch (e) {
        logger.error("❌ Exception while creating event: " + e);
        return null;
    }
}

🔍 4. Function: Find Event by Transaction ID

📌 Purpose: Retrieves an event by its Transaction ID (SCH-1) stored in categories[].
📌 Function Definition

/**
 * Finds an event in Microsoft Outlook by Transaction ID.
 *
 * @param {String} accessToken - OAuth2 access token
 * @param {String} userId - Microsoft Outlook user ID
 * @param {String} transactionId - Transaction ID (SCH-1)
 * @return {String} Event ID if found, otherwise null
 */
function findMicrosoftCalendarEventByCustomTransactionId(accessToken, userId, transactionId) {
    var filterQuery = "categories/any(a:a eq '" + transactionId + "')";
    var encodedFilterQuery = encodeURIComponent(filterQuery);
    var searchUrl = "https://graph.microsoft.com/v1.0/users/" 
        + encodeURIComponent(userId) 
        + "/events?$filter=" + encodedFilterQuery;

    var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
    var HttpGet = Packages.org.apache.http.client.methods.HttpGet;
    var EntityUtils = Packages.org.apache.http.util.EntityUtils;

    try {
        var httpClient = HttpClients.createDefault();
        var httpGet = new HttpGet(searchUrl);
        httpGet.setHeader("Authorization", "Bearer " + accessToken);
        httpGet.setHeader("Content-Type", "application/json");

        var response = httpClient.execute(httpGet);
        var responseString = EntityUtils.toString(response.getEntity(), "UTF-8");
        var responseJson = JSON.parse(responseString);

        if (responseJson.value && responseJson.value.length > 0) {
            return responseJson.value[0].id;
        } else {
            logger.error("❌ No matching event found.");
            return null;
        }
    } catch (e) {
        logger.error("❌ Exception while retrieving Event ID: " + e);
        return null;
    }
}

📝 5. Function: Delete Event

📌 Purpose: Deletes an event from Microsoft Outlook Calendar.
📌 Purpose: Deletes an event by its Transaction ID (SCH-1) stored in categories[].
📌 Function Definition

/**
 * Deletes a Microsoft Outlook calendar event.
 *
 * @param {String} accessToken - OAuth2 access token
 * @param {String} userId - Microsoft Outlook user ID
 * @param {String} eventId - Event ID to delete
 */
function deleteMicrosoftCalendarEvent(accessToken, userId, eventId) {
    if (!accessToken || !userId || !eventId) {
        logger.error("❌ Missing required parameters: accessToken, userId, or eventId.");
        return;
    }

    var deleteUrl = "https://graph.microsoft.com/v1.0/users/" + encodeURIComponent(userId) + "/events/" + encodeURIComponent(eventId);

    var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
    var HttpDelete = Packages.org.apache.http.client.methods.HttpDelete;
    var EntityUtils = Packages.org.apache.http.util.EntityUtils;

    try {
        var httpClient = HttpClients.createDefault();
        var httpDelete = new HttpDelete(deleteUrl);
        httpDelete.setHeader("Authorization", "Bearer " + accessToken.trim());
        httpDelete.setHeader("Content-Type", "application/json");

        var response = httpClient.execute(httpDelete);
        var statusCode = response.getStatusLine().getStatusCode();

        if (statusCode === 204) {
            logger.info("✅ Event Deleted Successfully!");
        } else {
            var responseString = EntityUtils.toString(response.getEntity(), "UTF-8");
            logger.error("❌ Error Deleting Event: " + responseString);
        }
    } catch (e) {
        logger.error("❌ Exception while deleting: " + e);
    }
}

📅 6. Function: Get Access Token (OAUTH)

📌 Purpose: Get's OAUTH Token for requesting access to Microsoft Outlook Calendar graph.

/**
	Modify the description here. Modify the function name and parameters as needed. One function per
	template is recommended; create a new code template for each new function.

	@param {String} clientId - arg1 description
	@param {String} clientId - arg1 description
	@param {String} clientId - arg1 description
	
	@return {String} return access token for oauth2
*/

function getMicrosoftAccessToken(clientId, clientSecret, tenantId) {

	var scope = "https://graph.microsoft.com/.default";  // 🔹 Graph API scope

	// Microsoft OAuth 2.0 Token URL
	var tokenUrl = "https://login.microsoftonline.com/" + tenantId + "/oauth2/v2.0/token"; 

	// Import Apache HttpClient classes
	var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
	var HttpPost = Packages.org.apache.http.client.methods.HttpPost;
	var StringEntity = Packages.org.apache.http.entity.StringEntity;
	var EntityUtils = Packages.org.apache.http.util.EntityUtils;

	
	try {
	   var httpClient = HttpClients.createDefault();
	   var httpPost = new HttpPost(tokenUrl);
	   httpPost.setHeader("Content-Type", "application/x-www-form-urlencoded");
	
	   // 🔥 Request body for client credentials grant
	   var postData = "client_id=" + encodeURIComponent(clientId) +
	                  "&client_secret=" + encodeURIComponent(clientSecret) +
	                  "&scope=" + encodeURIComponent(scope) +
	                  "&grant_type=client_credentials";
	
	   httpPost.setEntity(new StringEntity(postData, "UTF-8"));
	
	   var response = httpClient.execute(httpPost);
	   var responseString = EntityUtils.toString(response.getEntity(), "UTF-8");
	   var responseJson = JSON.parse(responseString);
//	   logger.info("response: " + responseString);
	   if (responseJson.access_token) {
//	       logger.info("✅ Access Token Received: " + responseJson.access_token);
	       return responseJson.access_token;
	   } else {
	       logger.error("❌ Error getting access token: " + responseString);
	       return null;
	   }
	} catch (e) {
	   logger.error("❌ Exception: " + e);
	   return null;
	}
}

📅 7. Function: Update Microsoft Calendar

📌 Purpose: Updates the calendar event Microsoft Outlook Calendar graph.
📌 Purpose: Updates an event by its Transaction ID (SCH-1) stored in categories[].


/**
	Modify the description here. Modify the function name and parameters as needed. One function per
	template is recommended; create a new code template for each new function.

	@param {String} accessToken - arg1 description
	@param {String} userId - arg1 description
	@param {String} eventId - arg1 description
	@param {String} updatedEventData - arg1 description
	
	@return {String} return description
*/
function updateMicrosoftCalendarEvent(accessToken, userId, eventId, updatedEventData) {

	// Import Apache HttpClient classes
	var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
	var HttpPost = Packages.org.apache.http.client.methods.HttpPost;
	var StringEntity = Packages.org.apache.http.entity.StringEntity;
	var EntityUtils = Packages.org.apache.http.util.EntityUtils;
	
    if (!accessToken || !userId || !eventId) {
        logger.error("❌ Missing required parameters: accessToken, userId, or eventId.");
        return;
    }

    var updateUrl = "https://graph.microsoft.com/v1.0/users/" + encodeURIComponent(userId) + "/events/" + encodeURIComponent(eventId);

    var HttpClients = Packages.org.apache.http.impl.client.HttpClients;
    var HttpPatch = Packages.org.apache.http.client.methods.HttpPatch;
    var StringEntity = Packages.org.apache.http.entity.StringEntity;
    var EntityUtils = Packages.org.apache.http.util.EntityUtils;

    try {
        var httpClient = HttpClients.createDefault();
        var httpPatch = new HttpPatch(updateUrl);

        httpPatch.setHeader("Authorization", "Bearer " + accessToken.trim());
        httpPatch.setHeader("Content-Type", "application/json");
        httpPatch.setEntity(new StringEntity(JSON.stringify(updatedEventData), "UTF-8"));

//        logger.info("📌 Sending PATCH request to: " + updateUrl);
//        logger.info("📤 Request Body: " + JSON.stringify(updatedEventData, null, 2));

        var response = httpClient.execute(httpPatch);
        var responseString = EntityUtils.toString(response.getEntity(), "UTF-8");

        if (response.getStatusLine().getStatusCode() === 200) {
            logger.info("✅ Event Updated Successfully!");
        } else {
            logger.error("❌ Error Updating Event: " + responseString);
        }
    } catch (e) {
        logger.error("❌ Exception while updating: " + e);
    }
}

📜 6. Licensing

📌This project is licensed under the [Server Side Public License (SSPL) v1.0](https://www.mongodb.com/licensing/server-side-public-license).

