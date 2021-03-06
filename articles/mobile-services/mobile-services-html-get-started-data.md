<properties 
	pageTitle="Add Mobile Services to an existing app (HTML 5) | Microsoft Azure" 
	description="Learn how to get started using Mobile Services in an existing HTML app." 
	services="mobile-services" 
	documentationCenter="" 
	authors="ggailey777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="mobile-services" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-html" 
	ms.devlang="javascript" 
	ms.topic="article" 
	ms.date="08/16/2015" 
	ms.author="glenga"/>

# Add Mobile Services to an existing app

[AZURE.INCLUDE [mobile-services-selector-get-started-data](../../includes/mobile-services-selector-get-started-data.md)]

##Overview 

This topic shows you how to use Azure Mobile Services to leverage data in an HTML app. In this tutorial, you will download an app that stores data in memory, create a new mobile service, integrate the mobile service with the app, and then sign in to the Azure Management Portal to view changes to data made when running the app.

>[AZURE.NOTE]This tutorial is intended to help you better understand how Mobile Services enables you to use Azure to store and retrieve data from an HTML app. As such, this topic walks you through many of the steps that are completed for you in the Mobile Services quickstart. If this is your first experience with Mobile Services, consider first completing the tutorial [Get started with Mobile Services].

> [AZURE.IMPORTANT] To complete this tutorial, you need an Azure account. If you don't have an account, you can create a free trial account in just a couple of minutes. For details, see [Azure Free Trial](http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A756A2826&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2documentation%2Farticles%2Fmobile-services-html-get-started-data).

###Additional requirements

You can host the GetStartedWithData app on any web server. However, for your convenience scripts have been provided that enable you to run the app on `http://localhost:8000`.
 
+ To use localhost, this tutorial requires that you have one of the following web servers running on your local computer:

	+  **On Windows**: IIS Express. IIS Express is installed by the [Microsoft Web Platform Installer].   
	+  **On MacOS X**: Python, which should already be installed.
	+  **On Linux**: Python. You must install the [latest version of Python]. 
	
	You can use any web server to host the app, but these are the web servers that are supported by the downloaded scripts.  

+ A web browser that supports HTML5.

##<a name="download-app"></a>Download the GetStartedWithData project

This tutorial is built on the [GetStartedWithData app], which is an HTML5 app. The UI for this app is identical to the app generated by the Mobile Services quickstart, except that added items are stored locally in memory. 

1. [Download the HTML app project files][GetStartedWithData app].

2. In an HTML editor, open the downloaded project and examine the app.js file.

   	Notice that added items are stored in an in-memory **Array** object (**staticItems**). Refresh the page, and the data disappears. It is not persisted.

3. Launch one of the following command files from the **server** subfolder.

	+ **launch-windows** (Windows computers) 
	+ **launch-mac.command** (Mac OS X computers)
	+ **launch-linux.sh** (Linux computers)

	> [AZURE.NOTE] On a Windows computer, type `R` when PowerShell asks you to confirm that you want to run the script. Your web browser might warn you to not run the script because it was downloaded from the internet. When this happens, you must request that the browser proceed to load the script
	
	This starts a web server on your local computer to host the new app.

4. Open the URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> in a web browser to start the app.

5. In the app, type meaningful text, such as _Complete the tutorial_, in **Enter new task**, and then click **Add**.

   	![][0]  

   	Notice that the saved text is added to the **staticItems** array, and the page is refreshed to display the new item.

##<a name="create-service"></a>Create a new mobile service in the Management Portal

[AZURE.INCLUDE [mobile-services-create-new-service-data](../../includes/mobile-services-create-new-service-data.md)]

##<a name="add-table"></a>Add a new table to the mobile service

To be able to store app data in the new mobile service, you must first create a new table in the associated SQL Database instance.

1. In the Management Portal, click **Mobile Services**, and then click the mobile service that you just created.

2. Click the **Data** tab, then click **+Create**.

   	This displays the **Create new table** dialog.

3. In **Table name** type _TodoItem_, then click the check button.

  	This creates a new storage table **TodoItem** with the default permissions set. This means that anyone with the application key, which is distributed with your app, can access and change data in the table. The table is created in a schema that is specific to a given mobile service. This is to prevent data collisions when multiple mobile services use the same database.

4. Click the new **TodoItem** table and verify that there are no data rows.

	>[AZURE.NOTE]New tables are created with the Id, __createdAt, __updatedAt, and __version columns. When dynamic schema is enabled, Mobile Services automatically generates new columns based on the JSON object in the insert or update request. For more information, see [Dynamic schema](http://msdn.microsoft.com/library/windowsazure/jj193175.aspx).

6. In the **Configure** tab, verify that `localhost` is already listed in the **Allow requests from host names** list under **Cross-Origin Resource Sharing (CORS)**. If it's not, type `localhost` in the **Host name** field and then click **Save**.


	> [AZURE.IMPORTANT] If you deploy the quickstart app to a web server other than localhost, you must add the host name of the web server to the **Allow requests from host names** list. For more information, see [Cross-origin resource sharing](http://msdn.microsoft.com/library/windowsazure/dn155871.aspx"%20target="_blank).

You are now ready to use the new mobile service as data storage for the app.

##<a name="update-app"></a>Update the app to use the mobile service for data access

Now that your mobile service is ready, you can update the app to store items in Mobile Services instead of the local collection. 

3. In the Management Portal, click **Mobile Services**, and then click the mobile service you just created.

4. Click the **Dashboard** tab and make a note of the **Mobile Service URL**, then click **Manage keys** and make a note of the **Application key**.

  	You will need these values when accessing the mobile service from your app code.

1. In your web editor, open the index.html project file and add the following to the script references for the page:

        <script src="http://ajax.aspnetcdn.com/ajax/mobileservices/MobileServices.Web-1.2.5.min.js"></script>

5. In the editor, open the file app.js, uncomment the following code that defines the **MobileServiceClient** variable, and supply the URL and application key from the mobile service in the **MobileServiceClient** constructor, in that order:

	    var MobileServiceClient = WindowsAzure.MobileServiceClient,
			client = new MobileServiceClient('AppUrl', 'AppKey'),   		    

  	This creates a new instance of MobileServiceClient that is used to access your mobile service.

6. Comment-out the following lines of code:

		var itemCount = 0;
		var staticItems = [];

	Data will be stored in the mobile service and not in an in-memory array.

6. Uncomment the following line of code:

        todoItemTable = client.getTable('todoitem');

   	This code creates a proxy object (**todoItemTable**) for the SQL Database **TodoItem**. 

7. Replace the **$('#add-item').submit** event handler with the following code:

		$('#add-item').submit(function(evt) {
			var textbox = $('#new-item-text'),
				itemText = textbox.val();
			if (itemText !== '') {
				todoItemTable.insert({ text: itemText, complete: false })
					.then(refreshTodoItems);
			}
			textbox.val('').focus();
			evt.preventDefault();
		});


  	This code inserts a new item into the table.

8. Replace the **refreshTodoItems** method with the following code:

		function refreshTodoItems() {

			var query = todoItemTable;

			query.read().then(function(todoItems) {
				listItems = $.map(todoItems, function(item) {
					return $('<li>')
						.attr('data-todoitem-id', item.id)
						.append($('<button class="item-delete">Delete</button>'))
						.append($('<input type="checkbox" class="item-complete">').prop('checked', item.complete))
						.append($('<div>').append($('<input class="item-text">').val(item.text)));
				});
					   
				$('#todo-items').empty().append(listItems).toggle(listItems.length > 0);
				$('#summary').html('<strong>' + todoItems.length + '</strong> item(s)');
			});
		}
	   

   This sends a query to the mobile service that returns all items. The results is iterated over and data is displayed on the page. 

9. Replace the **$(document.body).on('change', '.item-text')** and **$(document.body).on('change', '.item-complete')** event handlers with the following code:
        
		$(document.body).on('change', '.item-text', function() {
			var newText = $(this).val();
			todoItemTable.update({ id: getTodoItemId(this), text: newText });
		});

		$(document.body).on('change', '.item-complete', function() {
			var isComplete = $(this).prop('checked');
			todoItemTable.update({ id: getTodoItemId(this), complete: isComplete })
				.then(refreshTodoItems);
		});
 
   	This sends an item update to the mobile service when text is changed or when the box is checked.

10. Replace the **$(document.body).on('click', '.item-delete')** event handler with the following code:

		$(document.body).on('click', '.item-delete', function () {
			todoItemTable.del({ id: getTodoItemId(this) }).then(refreshTodoItems);
		});

	This sends a delete to the mobile service when the **Delete** button is clicked.

Now that the app has been updated to use Mobile Services for backend storage, it's time to test the app against Mobile Services.

##<a name="test-app"></a>Test the app against your new mobile service

4. Reload the URL <a href="http://localhost:8000/" target="_blank">http://localhost:8000/</a> in a web browser start the app.

    > [AZURE.NOTE] If you need to restart the web server, repeat the steps in the first section.

2. As before, type text in **Enter new task**, and then click **Add**. 

   	This sends a new item as an insert to the mobile service.

3. In the [Management Portal], click **Mobile Services**, and then click your mobile service.

4. Click the **Data** tab, then click **Browse**.
  
   	Notice that the **TodoItem** table now contains data, with id values generated by Mobile Services, and that columns have been automatically added to the table to match the TodoItem class in the app.

5. In the app, check one of the items in the list, then go back to the Browse tab in the portal and click **Refresh**. 

  	Notice that the complete value has changed from **false** to **true**.

6. In the app.js project file, locate the **RefreshTodoItems** method and replace the line of code that defines `query` with the following:

   		var query = todoItemTable.where({ complete: false });

7. Load the page again, check another one of the items in the list.

   	Notice that the checked item now disappears from the list. Each update results in a round-trip to the mobile service, which now returns filtered data.

This concludes the **Get started with data** tutorial.

## <a name="next-steps"> </a>Next steps

This tutorial demonstrated the basics of enabling an HTML app to work with data in Mobile Services. Next, learn how to authenticate users of your app try one of these other tutorials by completing [Add authentication to your app]. If you are working with a Cordova or PhoneGap app project, learn more about push notifications in [Push Notifications to Cordova Apps with Microsoft Azure](https://msdn.microsoft.com/magazine/dn879353.aspx).

<!-- Anchors. -->
[Download the HTML app project]: #download-app
[Create the mobile service]: #create-service
[Add a data table for storage]: #add-table
[Update the app to use Mobile Services]: #update-app
[Test the app against Mobile Services]: #test-app
[Next Steps]:#next-steps

<!-- Images. -->
[0]: ./media/mobile-services-html-get-started-data/mobile-quickstart-startup-html.png

<!-- URLs. -->
[Get started with Mobile Services]: mobile-services-html-get-started.md
[Add authentication to your app]: mobile-services-html-get-started-users.md

[Azure Management Portal]: https://manage.windowsazure.com/
[Management Portal]: https://manage.windowsazure.com/
[GetStartedWithData app]:  http://go.microsoft.com/fwlink/?LinkID=286345

[Mobile Services HTML/JavaScript How-to Conceptual Reference]: mobile-services-html-how-to-use-client-library.md

[Cross-origin resource sharing]: http://msdn.microsoft.com/library/azure/dn155871.aspx

 

test
