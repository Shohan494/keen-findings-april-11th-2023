# conocido-monthly-all-bills-status-11-april-2023

#### started with keen, better understand the flow first,

- using mapi api, the real api, imported list of customers in cust_refs_new

```	
  public function importLatestCustRefs()
	{
		include(app_path() . '/cdbackend/mapi.php');
		
		$m = new Mapi('demouser','demopw');
		
		$customers = $m->ListCustomers('', 1, 100);  // Find all customers with 'janssen' in their name, first page, pagesize = 10
		
		DB::table('cust_refs_new')->truncate();
		
		foreach($customers as $customer)
		{
			$insertCustomerdata = DB::table('cust_refs_new')->insert([
				'companyname'	=>    $customer->companyname,
				'referencecode'	=>    $customer->referencecode
			]);
		}
	}

```
- okay, later on using keen php api auth and we import details about customer, which is saved to table named current_customer_data and target is to have the customerId, example "16536"
```
$getcurrentcustomerdata = $keenConnectUser->QueryBackend('getcurrentcustomer');
```
- for importing invoices, we need the customerId
```
$invoiceList = $keenConnectUser->QueryBackend('listinvoices');
```
- update was done basically when I did save the local keen table,
but now the status update was changed on based on api response when it is called
- KeenImportInvoiceAndSendingToInvoicehubController -> automatedCallForKeenSystem -> sendPaymentDataToInvoicehub --> invoicehub api call response --> then status update --> DB::table('list_invoices')->where('invoicenr', $invoices->invoicenr)->update(['invoicehub_status' => "sent"]);

- =============================================
- in invoicehub we have to sure latest current_customer_data, from api call or in any way, *THIS IS A TASK THAT SHOULD BE DONE*
- invoicehub to hostfact should not be a problem afterwards

- for march data, 202303, ensured keen to invoicehub current_customer_data import then processed for hostfact
- here came something, some current_customer_data table's field named referencecode contains different debtorcode structure, we have to fix them for errorless hostfact invoicing, example : DB0064_TWC means DB0064, DB0064 also means DB0064, ZDGHDB0064 does also mean DB0064
- for the above issue, manual or a script solution both are applicable, **but if we do import again and again, script is better solution**





#### Process findings on 11th april, 2023

- error 1st time pw mismatch api, reason file last update october, 2022, this always happens, and we may lose enough code even elsewhere, *SHOULD BE A CONCERN TO ENSURE PREVENTING THIS*

- error 2nd mail limit - Expected response code 250 but got code "554", with message "554 5.7.0 Your message could not be sent. 
The limit on the number of allowed outgoing messages was exceeded. Try again later. "

- keen to api invoicehub sending limit - 9 ( [message] => Data Inserted Successfully! - based on output text COUNT), 
right after then the below query was executed ( status sent and period 202303 )
result : Showing rows 0 - 19 (20 total, Query took 0.0010 seconds.) [id: 983... - 964...] 

- total import keen data row  Showing rows 0 - 24 (34 total, Query took 0.0004 seconds.) [id: 997... - 973...]

- 3rd time run, same error - Expected response code 250 but got code "554", with message "554 5.7.0 Your message could not be sent. The limit on the number of allowed outgoing messages was exceeded. Try again later. "

## till now the relavent screesnshots and also keen automation file uploaded after edit on 11th april, 2023: 

- Capture.PNG
- Capture2.PNG
- Capture3.PNG

- keen file after update 11th april 2023 for future check (git commit)

## after break

- commented mail
- ran the cron
- Data Inserted Successfully! count 14 and

-  Showing rows 0 - 24 (34 total, Query took 0.0003 seconds.) [id: 997... - 973...], Query is :
SELECT * FROM `list_invoices` WHERE `invoicehub_status` LIKE 'sent' AND `invoiceperiod` = 202303 ORDER BY `id` DESC 

- all data sent to invoicehub at 05:37 pm

- ???? from invoicehub question arises, conversation :

```
I have a question
keen customer data reference value
Does it change? I mean
update?
Yes and no
So the relation would usually never change
But we should signal new customers in KeenSystems and check for their reference
And process that data
Asking this, cause important, if the reference is changed or updated, then there is a possibility of that we will create wrong invoices
if no update or change, we don't have to bother
that's all
cause basically it's the debtor code
as you know I believe
I did get a report from a customer that they got februari invoiced twice
We really need to improve this process
I have to check it out still
I have figured out all the issues today
and noted down
Whatever, in my opinion, we can add some more code, to take latest data of keen customers, and then process
```




