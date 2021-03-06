# Monitoring the Power Platform: Power Apps - Power Apps Component Framework

## Summary

The idea of monitoring **Dynamics 365 or Model Driven Applications** is not a new concept. Understanding where services are **failing**, users are **interacting** with the platform, where **form** and **business processes** could be **tuned for performance** are key drivers for most if not all businesses, from small companies to enterprises. Luckily, the **Dynamics 365** platform provides many tools to help audit and monitor business and operational events.

This article will cover adding **Azure Application Insights** functionality to **Power Apps Component Framework** controls. In this article we will briefly touch on the subjects of **Power Apps Component Framework** and **NPM packages**. We will explore building and extending a sample **Power Apps Component Framework** control to send events to **Azure Application Insights**. We will conclude with context considerations and reviewing events. 

## What is Power Apps Component Framework?

["Power Apps component framework empowers professional developers and app makers to create code components for model-driven and canvas apps (public preview) to provide enhanced user experience for the users to work with data on forms, views, and dashboards."](https://docs.microsoft.com/en-us/powerapps/developer/component-framework/overview) 

**Power Apps Component Framework Controls** can be **added to both Canvas and Model Driven Applications to extend the user experience** beyond the standard controls available. Examples include grids behaving like maps, number controls turned into dials or sliders and even integrations with **Azure Cognitive services**.

**Power Apps Component Framework Controls** render in context of the **Model Driven Application** form unlike web resources and as such behave like other controls. Multiple **Power Apps Component Framework** controls can be added to a form. **Power Apps Component Framework** controls are solution aware and can be added and migrated across environments just like other components of a solution.

## Application Insights JavaScript GitHub Repo

[The official **Azure Application Insight**s JavaScript GitHub Repository](https://github.com/microsoft/ApplicationInsights-JS) contains all the information needed to download and install the SDK for use within a **Power Apps Component Framework** control. [The Getting Started section](https://github.com/microsoft/ApplicationInsights-JS#getting-started) describes creating an **Azure Application Insights** resource and installing the correct **NPM package**.

## Application Insights NPM Package

To install the **Azure Application Insights NPM package**, use the following command (I use **Visual Studio Code**'s terminal):

```
npm i --save @microsoft/applicationinsights-web
```

## Implementing Application Insights in a PCF Control

This section assumes you have already covered the basics for creating a **PCF control**. If not, please refer to the official documentation "". [The next two sections can also be found in the Azure Application Insights JavaScript SDK GitHub repo](https://github.com/microsoft/ApplicationInsights-JS#basic-usage). I've adjusted the code slightly **to take into account the way PCF controls render**.

### Add Application Insights Dependencies

Next, in the **index.ts** file, we need to import the module:

```
import { ApplicationInsights, IEventTelemetry } from '@microsoft/applicationinsights-web'
```

The next step is to add an instance of the **ApplicationInsights** object. I added mine to the **PCF control class that allowed for use across the different methods used by PCF**. That said you may choose to implement **Azure Application Insights** in another way, such as in the global namespace, so where and how you decide to create this object is up to the developer.

```
  pcfControlAppInsights = new ApplicationInsights({ config: {
	instrumentationKey: '<your instrumentation key>',
	enableResponseHeaderTracking: true,
	enableRequestHeaderTracking: true
	/* ...Other Configuration Options... */
   } });
```

As shown above, various configuration properties can be set here. These include **how often messages are sent, how long sessions last, if sampling should be used**, etc. I'm including request and response header tracking in this example, but many others exist. [For additional properties, review the **ApplicationInsights-JS** GitHub documentation](https://www.bing.com/search?q=learn+PCF+control). 

### Initialize Application Insights

Once the **ApplicationInsights** object has been created for use, the first step is to initialize the instance.

```
this.pcfControlAppInsights.loadAppInsights();
```

The **Azure Application Insights** module only **expects to be loaded once**, attempting to do so again could **result in an error**. 

```
Error: Core should not be initialized more than once
    at AppInsightsCore.BaseCore.initialize (webpack://pcf_tools_652ac3f36e1e4bca82eb3c1dc44e6fad/./node_modules/@microsoft/applicationinsights-core-js/dist-esm/JavaScriptSDK/BaseCore.js?:46:13)
    at AppInsightsCore.initialize (webpack://pcf_tools_652ac3f36e1e4bca82eb3c1dc44e6fad/./node_modules/@microsoft/applicationinsights-core-js/dist-esm/JavaScriptSDK/AppInsightsCore.js?:33:33)
    at Initialization.loadAppInsights (webpack://pcf_tools_652ac3f36e1e4bca82eb3c1dc44e6fad/./node_modules/@microsoft/applicationinsights-web/dist-esm/Initialization.js?:299:16)
```

To account for this, the core includes a ***isInitialized*** method to help determine is the SDK has been loaded. To extend the previous example, **include it within a conditional check**:

```
   if (!this.pcfControlAppInsights.core.isInitialized!()){
	//Use this to load the app insights dependencies for use.
	this.pcfControlAppInsights.loadAppInsights();
    }
```

**Once loaded, the Application Insights object can be used immediately** to record page views, exceptions or any event that needs to be documented. To begin using and seeing an example, simply add a call to the ***trackPageView*** method which requires zero inputs.

```
//Send Page View Event
this.pcfControlAppInsights.trackPageView();
```

### Usage across PCF Control Methods

**PCF Controls** have three main methods used within the lifecycle of a control consisting of the initialization of the control (***init***), changes to the control (***updateView***) and removal of the control (***destroy***). The usage of **PCF controls** vary greatly and other methods can be used for input output, HTML elements within the control, etc which will not be covered here. [If you'd like to learn more, there are many great tools and open source projects that go into this at great length.](https://www.bing.com/search?q=learn+PCF+control) 

For this article just be aware that **Azure Application Insights** can be used to track virtually any callback with a control. Examples include **user clicks of a button, responses from API calls**, or as described above, **when and how a control is created or initialized**.

Here is an example of tracking a custom event within the **initialize** method:

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PcfControl.init.JPG" style="zoom:50%;" />

Here is an example of tracking a custom event within the **update view** method:

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PcfControl.updateView.JPG" style="zoom:50%;" />

Here is an example of tracking a custom event within the **get outputs** method:

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PcfControl.getOutputs.JPG" style="zoom:50%;" />

Here is an example of tracking a custom event within the **destroy** method:

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PcfControl.Destroy.JPG" style="zoom:50%;" />

## Configure Context

Adding context to **Azure Application Insights** messages is extremely important and vital to leveraging the robust feature set associated with the service. That said, out of the box, the SDK provides automatic context properties and allows developers the ability to overwrite. This provides the potential to select well known values or correlating identifiers such as form or entity Ids in **Dynamics 365**, system user Ids, **Model Driven Application** Ids and so on.

The image below shows a captured Fiddler trace of a default context properties.

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.DefaultContext.Fiddler.JPG"  />

[To add or modify context for all messages](https://github.com/microsoft/ApplicationInsights-JS#telemetry-initializers) sent to **Azure Application Insights**, use the ***addTelemetryInitializer*** method. This method provides the ability to pass in a custom envelope which can be extended to modify or extend properties of an Azure Application Insights message. Here, developers can set cloud role and instance, operation names and identifiers, and even custom properties that will be included in each record in the **Azure Application Insights** tables.

 <img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PcfControl.AddTelemetryInitalizer.Full.JPG" style="zoom:50%;" />

The first part of the code above creates an object that contains **Azure Application Insights** tags and custom properties. These **tags will align with the session and operational context for each message** sent from the PCF control.

```
var telemetryInitializer = (envelope) => {
	envelope.tags["ai.cloud.role"] = orgSettings._customControlExposedOrgSettings.organizationId;
	envelope.tags["ai.cloud.roleInstance"] = response;
	envelope.tags["ai.session.id"] = window.URL;
	envelope.tags["ai.operation.name"] = QnAMakerControl.name;
	envelope.data.cloudRole = 'just checking in';
	}
```

In the sample code above the **cloud role and instance** are set to the **Dynamics 365** organization and **Model Driven Application** identifier. The values used here will impact how the **Application Map in Azure Application Insights renders**. 

The **operation name** is the class name of the **PCF control**. Operations include both the core operation and the parent, this could be thought of as **<u>the core operation being the event (initialization of the control) and the parent being the workblock (loading of the Dynamics 365 form)</u>**.

Next, add the **ITelemetryItem** created above using the [addTelemetryInitializer](https://github.com/microsoft/ApplicationInsights-JS/blob/master/API-reference.md#addTelemetryInitializer) method:

```
//Add the context properties
this.pcfControlAppInsights.addTelemetryInitializer(telemetryInitializer);
```

To reiterate, setting the context, while desired for many reasons, is completely optional. Following the Implementing **Application Insights** in a **PCF control** section above will work just fine.

At this point session, role and operation data points have been set, one other point I'd recommend reviewing is adding the user identifier. This can be added as a custom property within the custom dimensions property bag but for this example we will use the ***Authenticated User*** property on the core **ApplicationInsights** object.

```
//Set user authenticated
this.pcfControlAppInsights.setAuthenticatedUserContext(context.userSettings.userId, undefined, true);
```

The sample code above uses the [***userId***](https://docs.microsoft.com/en-us/powerapps/developer/model-driven-apps/clientapi/reference/xrm-utility/getglobalcontext/usersettings#userid) provided from the context for **Model Driven Applications**. Consider a different approach if working with **Canvas Driven Applications**.

## Reviewing Application Insights

Once the control is implemented with a **Model Driven Application**, data will being to flow to **Azure Application Insight**s. An interesting note here is that the **Azure Application Insights SDK** is also used by the platform. In some cases the events sent from your **Model Driven Application** may be delivered in the same envelope! Don't worry, this has no impact to the platform and will not break if the platform changes how it delivers messages.

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.HitchingARide.JPG" style="zoom:80%;" />

Once delivered, **user session and events** will begin to appear. The below example shows the **init** method, followed by the **updateview** and **getOutputs** with user interaction in between.

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PCFControl.Session.Full.JPG" style="zoom:80%;" />

Example of a **custom event** record:

<img src="https://raw.githubusercontent.com/aliyoussefi/MonitoringPowerPlatform/master/Artifacts/PCF/AppInsights.PCFControl.CustomEvent.Full.JPG" style="zoom:80%;" />

[This reference includes all the sample code above and additional usage](https://github.com/aliyoussefi/MonitoringPowerPlatform/blob/master/Samples/PCF/AppInsights/index.ts) showing how to send various events to **Azure Application Insights**.

## Next Steps

In this article we have discussed how to add **Azure Application Insights** to **Power Apps Component Framework** controls. Continue exploring and evaluating how the initialization of the **ApplicationInsights** object can be extended with different properties, such as **<u>enableAutoRouteTracking</u>**

In an upcoming article, we will explore the tools within **Azure Application Insights** that rely on the context properties discussed at the end of this current article. Configuring both the actual delivery mechanism and adding context will help expose insights from the data collected.

If you are interested in learning more about [**specialized guidance and training** for monitoring or other areas of the Power Platform](https://community.dynamics.com/crm/b/crminthefield/posts/pfe-dynamics-365-service-offerings), which includes a [monitoring workshop](https://community.dynamics.com/cfs-file/__key/communityserver-blogs-components-weblogfiles/00-00-00-17-38/WorkshopPLUS-_2D00_-Dynamics-365-Customer-Engagement-Monitoring-with-Application-lnsights-1-Day-with-Lab_2D00_FA5D599F_2D00_20E4_2D00_4087_2D00_A713_2D00_39FBD14DF7E5.pdf), please contact your **Technical Account Manager** or **Microsoft representative** for further details. 

Your feedback is **<u>extremely valuable</u>** so please leave a comment below and I'll be happy to help where I can! Also, if you find any inconsistencies, omissions or have suggestions, [please go here to submit a new issue](https://github.com/aliyoussefi/MonitoringPowerPlatform/issues).

## Index

[Monitoring the Power Platform: Introduction and Index](https://community.dynamics.com/crm/b/crminthefield/posts/monitoring-the-power-platform-introduction#index)
