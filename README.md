# SCVMMBilling
## Overview
* this app is for collecting Data From scvmm and calculating the data to make a price for each Vm depending on the Resource Usage and config
## Calling the App 
To run the App just Call the console app  ScvmmBilling.Console.exe and it will collect the data and check's if it's the correct time to generated bills 
* ### Main Args
    * ```date:year-month```: this arg is used to be able to get the bill of a specific past month given the year and the month 
    * Ex: ```ScvmmBilling.Console.exe date:year-month```
    
    *  ```test``` : this arg is used to test the app (its uses dummy data to check if the connection to the database is correct and the data is stored correcty)
    * Ex:  ```ScvmmBilling.Console.exe test```
    
    *  ```daily``` : get's the current price that a Vm would pay at this point of time for a day .
    * Ex:  ```ScvmmBilling.Console.exe daily -day-date:value```
    #### daily Args 
    - ```-day-date:value```  "value" can be either a day date Ex : -day-date:2021-05-09 (year-month-day) or "now" Ex : -day-date:now.
      * ```-day-date:2021-05-09``` get's you the bill for that exact day from 00:00:00 to 23:59:59
      * ```-day-date:now``` get's you the bill for that exact day from the start of the day to that time whenyou start the app



    - ``` history -client:value1 -date:value1 ``` is ued used to get the client's Daily usage of the resources of each virtual machine through out the month.
      - ```-client:value1``` Value1 would be the client's name
      - ```-day-date:value2``` value2 would be the date of the month you wanna get the result for.

    *Note that those args can't and should not be combined with eachother*
    *ex :```ScvmmBilling.Console.exe daily test``` is not allowed and will thorw an error message*
        
    
* ### Aditional args 
    * ```-no-email``` : used for testing when you want the app to not send emails just generate the files 
    *the ```-no-email``` arg can be combined with the above args*   
    
    *Ex : ```ScvmmBilling.Console.exe date:year-month -no-email``` (will generate the bill for that month and save them in a folder specified in the App.config but will not send them via Email)*

    * ``` -email-to:desiredEmail ``` : used to override the default email to send bills to for a single execution "in case you want a specific month's bill to be sent to a specific email"
    *the ```-email-to:desiredEmail``` arg can be combined with the above args* 

    * ``` -keep-files ``` : used to tell the app no to delete the Generated files after sending them via email."
    *the ``` -keep-files ``` arg can be combined with the above args* 

    *Ex : ```ScvmmBilling.Console.exe date:year-month -email-to:desiredEmail``` (will generate the bill for that month and save them in a folder specified in the App.config and will send the bill to the "desiredEmail").*

    *Note : ``` -no-email``` cancels the effect of ```-email-to``` or any email Function due to the fact that ```-no-email``` disables the Email Sending Mechanism. so even if you change emails or override them they will not be sent.*   
## configuration 

* Before Using this app some configurations might be neeeded for it to run:

1. First of all is the conection string:
    ```
        <connectionStrings>
            <add name="ConnectionString" connectionString="Server=HDCETS\SQLEXPRESS;Database=HdceAutomation;Trusted_Connection=True" />
        </connectionStrings>
    ```
2. Then is you have some Vms that need to be ignored in the billing proccess we have a config section for it.
    ```
        <ExcludeVmConfiguration>
            <VMs>
                <VM>
                    <VirtualMachine>
                        <Value name="test-vm" />
                    </VirtualMachine>
                </VM>
                <VM>
                    <VirtualMachine>
                        <Value name="test-vm1" />
                    </VirtualMachine>
                </VM>
            </VMs>
        </ExcludeVmConfiguration>
    ```
3. Then the Prices foreach Resource Can be configured too 
    * this is devided to 2 sections: 
        * first is default prices: those get's applied for everything 
            ``` 
                <defaultPricingConfiguration>
                    <defaults>
                        <default ram="00.00" cpu="00.00" disk_per_gygabyte="00.00" currency="CAD"/>
                    </defaults>
                </defaultPricingConfiguration> 
            ```
        * second is client prices: those get's apllied for a specific client specified in the config file.
            ```
                <clientsPricingConfiguration>
                    <clientPricing>
                        <clientPrices name="client1">
                            <prices>
                                <price ram="00.00" cpu="00.00" disk_per_gygabyte="00.00" currency="CAD"/>
                            </prices>
                        </clientPrices>
                        <clientPrices name="client2">
                            <prices>
                                <price ram="00.00" cpu="00.00" disk_per_gygabyte="00.00" currency="CAD"/>
                            </prices>
                        </clientPrices>
                    </clientPricing>
                </clientsPricingConfiguration>
            ```
4. Then Configuring the MailSettings to send mails .
    ```
        <MailSettings>
            <mailHosts>
                <settings mail="smtprcpem@smtp.local" displayname="HDCE Inventory check" host="smtpload.hdce.ca" password="000000" port="25"/>
            </mailHosts>
        </MailSettings>
    ```
5. Configure the Day to send Notification Email

    The app once a week sends a nottification email to an email adress when  the vm name don't follow some naming criteria.
    you can also configure the day you want this email to be sent using the following .
    In the appSettings section you'll find 
    ```
        <!-- Days are numbered From 0 to 6 while 0 is Sunday, Every number lessthan 0 or greaterthan 6 will be considered a 0 -->
        <add key="day" value="5"/>
    ```

6. the the last thing is the rules to follow when calculating the bill for each Vm status
    
    ```
        <BillingRulesConfiguration>
            <Rules>
                <Rule state="the state of the machine" rule="the rulle to follow"/>
                <Rule state="the state of the machine" rule="the rulle to follow"/>
            </Rules>
        </BillingRulesConfiguration>
    ``` 
    * Rules Can be: 
        ```
            space : to bill only the disk space 
            nospace :  to bill everything but not the space 
            Ram : to bill the ram only
            Cpu : to bill only the cpu
            all : bill the whole configuration
        ```
    * the states of the machine are : 
        ```
            Running, Stopped, Incomplete_VM_Configuration, Unsupported_Cluster_Configuration, Missing, Host_Not_Responding, Unsupported_Undo_Disk
        ``` 