# Telus IoT Starter Kit Walkthrough: Part 3

This is part 3 of a 3-part tutorial that will help get you started with the TELUS LTE-M IoT Starter Kit:
* **[Part 1](https://github.com/briantan050/Telus-IoT-Starter-Kit-Walkthrough-Part-1/)** will give you some background on the kit and walk you through the process of getting the kit configured to send data to your own Microsoft Azure instance.
* **[Part 2](https://github.com/briantan050/Telus-IoT-Starter-Kit-Walkthrough-Part-2/)** will walk you through using the IoT data in a logic app with the Copernicus open access hub API. 
* **[Part 3](https://github.com/briantan050/Telus-IoT-Starter-Kit-Walkthrough-Part-3/)** will walk you through displaying the IoT data in a Power BI dashboard.

### Displaying IoT data in a Power BI report
Now that you have sensor data being sent to your Azure IoT Hub, it would be useful to be able to display it in a way that can be easily understood by the user. Dashboards are tools to provide views of data that can update automatically. This tutorial will walk you through the process of creating a dashboard in Power BI to display the sensor's temperature and humidity data using line graphs, and the GPS coordinates on a map.

The list of steps is as follows:
* Configuring Azure
* Configuring Power BI

**IMPORTANT NOTE**: The Stream Analytics job may deplete the user's free credits and eventually cost additional funds to maintain. It must be closely monitored to make sure that it does not incur unwanted fees.

![Power_BI_dashboard](https://user-images.githubusercontent.com/53897474/158296508-7f430d96-0576-4017-bcd5-acc24c4dd862.png)

### Requirements
1. [Telus IOT Starter Kit](https://www.avnet.com/shop/us/products/avnet-engineering-services/aes-bg96-iot-sk2-g-3074457345636408150?INTCMP=tbs_low-power-wide-area_button_buy-your-kit)
2. [Microsoft Azure Account](https://azure.microsoft.com/en-ca/)
3. [Microsoft Power BI Account](https://powerbi.microsoft.com/en-ca/)

**Important note**: It is imperative that you use the same email for both the Microsoft Power BI Account and the Microsoft Azure account for this project. Linking the data from the Azure IoT Hub to the Power BI dashboard will only work if the same email is used for both accounts. Please test to make sure that you are able to make both accounts with the same email before starting **Part 1**. Register with [this link](https://powerbi.microsoft.com/en-ca/) (may require a work-email to register). 

# Configuring Azure
### Add a consumer group to your IoT hub
Consumer groups provide independent views into the event stream that enable apps and Azure services to independently consume data from the same Event Hub endpoint. In this section, you add a consumer group to your IoT hub's built-in endpoint that is used later in this tutorial to pull data from the endpoint.

To add a consumer group to your IoT hub, follow these steps:

1. In the Azure portal, open your **IoT hub**.
2. On the left pane, select **Built-in endpoints**. Enter a name for your new consumer group in the text box under Consumer groups.
![image](https://user-images.githubusercontent.com/53897474/158875567-0ac1e403-10d7-4651-b5cf-64ba0ad142aa.png)
3. Click anywhere outside the text box to save the consumer group.

### Create a Stream Analytics job
Stream Analytics jobs allow us to grab an **input**, **process** it with a query, and send an **output** to a specified location. In this case, we will be grabbing the sensor data sent by the Nucleo board (**input**), **processing** it with a SQL query, and then sending it to Power BI (**output**) to be displayed on a Power BI report.

1. In the Azure portal, select **Create a resource**. 
![image](https://user-images.githubusercontent.com/53897474/158875955-2c17f1c8-20d5-4388-86e2-4da128a7832f.png)

2. Type **Stream Analytics Job** in the search box and select it from the drop-down list. 
3. On the Stream Analytics job overview page, select **Create**
4. Enter the following information for the job.  

   **Job name**: The name of the job. The name must be globally unique.  
   
   **Resource group**: Use the same resource group that your IoT hub uses.  
   
   **Location**: Use the same location as your resource group.  
  
![image](https://user-images.githubusercontent.com/53897474/158876258-30fdb7c0-a42f-4ff2-b2bc-07bd4cbc0215.png)

5. Select **Create**.  

### Add an input to the Stream Analytics job
1. Open the Stream Analytics job.
2. Under Job topology, select **Inputs**.
3. In the Inputs pane, select **Add stream input** and select **IoT Hub** from the drop-down list. 
![image](https://user-images.githubusercontent.com/53897474/158876858-55fb7df0-b88c-4a71-95f5-cb83ad22ff57.png)

4. On the new input pane, enter the following information:  

    **Input alias**: Enter a unique alias for the input.  
    
    **Select IoT Hub from your subscription**: Select this radio button.  
    
    **Subscription**: Select the Azure subscription you're using for this tutorial.  
    
    **IoT Hub**: Select the IoT Hub you're using for this tutorial.  
    
    **Endpoint**: Select Messaging.  
    
    **Shared access policy name**: Select the name of the shared access policy you want the Stream Analytics job to use for your IoT hub. For this tutorial, you can select service. The service policy is created by default on new IoT hubs and grants permission to send and receive on cloud-side endpoints exposed by the IoT hub. To learn more, see Access control and permissions.  
    
    **Shared access policy key**: This field is autofilled based on your selection for the shared access policy name.  
    
    **Consumer group**: Select the consumer group you created previously.  
    
    **Leave all other fields at their defaults.**  
  
![image](https://user-images.githubusercontent.com/53897474/158876941-c6ed2353-a033-4071-bcd0-f93f11eaf7c9.png)

5. Select **Save**.

### Add an output to the Stream Analytics job
1. Under Job topology, select **Outputs**.

2. In the Outputs pane, select **Add**, and then select **Power BI** from the drop-down list.

3. On the Power BI - New output pane, select **Authorize** and follow the prompts to sign in to your Power BI account.

4. After you've signed in to Power BI, enter the following information:

    **Output alias**: A unique alias for the output.  

    **Group workspace**: Select your target group workspace.  

    **Dataset name**: Enter a dataset name.  

    **Table name**: Enter a table name.  

    **Authentication mode**: Leave at the default.  
  
![image](https://user-images.githubusercontent.com/53897474/158877010-e8f974e2-545e-4c12-ad58-dc6240453b56.png)

5. Select **Save**.

### Configure the query of the Stream Analytics job
Even though the data payloads can be sent and received successfully, the data will still arrive as text strings, making it impossible to display as numeric data on charts. We will therefore need to run a SQL query to process some of the data so that it can be sent to the Power BI report in a useable data format. In this step, we will use the `CAST()` function to convert the relevant variables into `float` format. 

1. Under Job topology, select **Query**.

2. Replace the SQL query with the following:  
  
```
SELECT
    ObjectName,
    ObjectType,
    CAST(Version AS float) as Version,
    ReportingDevice,
    CAST(Latitude AS float) as Latitude,
    CAST(Longitude AS float) as Longitude,
    CAST(GPSTime AS float) as GPSTime,
    CAST(GPSDate AS float) as GPSDate,
    CAST(Temperature AS float) as Temperature,
    CAST(Humidity AS float) as Humidity,
    CAST(Pressure AS float) as Pressure,
    CAST(Tilt AS float) as Tilt,
    CAST(ButtonPress AS float) as ButtonPress,
    TOD,
    EventProcessedUtcTime,
    PartitionId,
    EventEnqueuedUtcTime,
    IoTHub
INTO
    [YourOutputAlias]
FROM
    [YourInputAlias]
```
  
3. Replace `[YourInputAlias]` with the input alias of the job.

4. Replace `[YourOutputAlias]` with the output alias of the job.
![image](https://user-images.githubusercontent.com/53897474/158877055-ef16eddf-be7f-4f98-b33a-e9147db0d9ac.png)

### Run the Stream Analytics job
1. In the Stream Analytics job, select **Overview**, then select **Start > Now > Start**. 
2. Once the job successfully starts, the job status changes from Stopped to Running.
![image](https://user-images.githubusercontent.com/53897474/158877111-13679f4e-dcfd-4b47-95d8-881ff0527b09.png)

3. Start your sensor board and let it run until data payloads have been sent. You can keep track of this using the **Monitoring Payloads sent to Azure** section of this walkthrough. 
4. Navigating back to the **Query** section of the Stream Analytics Job, you will be able to see the incoming payloads being received. 
![image](https://user-images.githubusercontent.com/53897474/158877161-71d2e3b2-3c1e-46eb-9122-d0e87678cd36.png)

# Configuring Power BI
### Create a Power BI report
A Power BI report displays data from your dataset in a layout that you can design and configure yourself. We will be using the Power BI report to create a dashboard to display the sensor data.

1. Sign in to your Power BI account and select Power BI service from the top menu.
2. Select the workspace you used from the side menu, **My Workspace**.
4. Under the **All** tab, you should see the dataset that you specified when you created the output for the Stream Analytics job.
5. Hover over the dataset you created, select More options menu (the three dots to the right of the dataset name), and then select **Create report**.
![image](https://user-images.githubusercontent.com/53897474/158877208-0b146853-4a77-4c75-a023-02207ff16d5c.png)

### Configure a Power BI report to visualize the data
1. Select charts, tables and maps from the **Visualizations** menu to design your dashboard.
![image](https://user-images.githubusercontent.com/53897474/158877256-101ead23-694a-4bc7-ae30-c4c57e9d7ff2.png)

2. As an example, the configuration for the Line Chart Visualization is as follows:
* drag **Temperature** into the **Values** section, and **EventEnqueuedUtcTime** into the **Axis** section.
![image](https://user-images.githubusercontent.com/53897474/158877287-70cbca69-710f-44b6-b58d-22452e11f07d.png)

### Share the report
1. Select **Save** to save the report. When prompted, enter a name for your report. When prompted for a sensitivity label, you can select **Public** and then select **Save**.
![image](https://user-images.githubusercontent.com/53897474/158877321-81287cd1-5d5d-40b9-ad97-b4413e901bff.png)

2. Still on the report pane, select File > Embed report > Website or portal.
![image](https://user-images.githubusercontent.com/53897474/158877347-64dad571-a426-473c-9943-910b8c83ae5d.png)

* NOTE: If you get a notification to contact your administrator to enable embed code creation, you may need to contact them. Embed code creation must be enabled before you can complete this step.

3. You're provided the report link that you can share with anyone for report access and a code snippet that you can use to integrate the report into a blog or website. Copy the link in the Secure embed code window and then close the window.
![image](https://user-images.githubusercontent.com/53897474/158877389-de607315-3bf2-4fc4-a52e-e565c06cf0b9.png)

4. Open a web browser and paste the link into the address bar.
![image](https://user-images.githubusercontent.com/53897474/158877428-24f29f1a-97a6-4b87-918a-994cb999b173.png)

## Done
Your dashboard is now displaying sensor data from your Azure IoT Hub. In this tutorial, you have completed the following:
* Created a consumer group in your Azure IoT hub.
* Created and configured an Azure Stream Analytics job to read temperature telemetry from your consumer group and sent it to Power BI.
* Configured a report for the data in Power BI and shared it to the web.

## Credits:
* GarettB's tutorial: [TELUS IOT Getting Started](https://github.com/garettB/TELUS_IoT_Getting_Started)
* Microsoft Azure's tutorial: [Visualize real-time sensor data from Azure IoT Hub using Power BI](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-live-data-visualization-in-power-bi)
* Dinusha Kumarasiri's tutorial: [End to end IoT Solution with Azure IoT Hub, Event Grid and Logic Apps](https://youtu.be/Wb_QT0qHGOo)
* Reza Vahidnia and F. John Dian's book: [Cellular Internet of Things for Practitioners](https://pressbooks.bccampus.ca/cellulariot/)
