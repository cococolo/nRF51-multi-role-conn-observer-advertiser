# TIMESLOT ADVERTISER
This project utilizes the Concurrent Multiprotocol Timeslot API in the Nordic Semiconductor S110 Softdevice for the   nRF51-series microcontrollers to run a scannable advertiser concurrently with a Softdevice powered BLE Device.

The Timeslot advertiser accepts a subset of the HCI command interface to control various parameters of its behaviour.

## Example code
The project contains an example of usage, where the Timeslot advertiser runs with an advertisement interval of 100ms,
along with a Softdevice powered connectable advertiser with an advertisement interval of 150ms. 

## Timeslot advertiser: Interface
The timeslot advertiser interface is based on the Bluetooth standard HCI interface, but only implements a small subset of the functionality. All interface functions are found in the ts_advertiser.h file under include/.

### Report queue
The timeslot advertiser offers reports related to radio activity to the user. The software interrupt indicated by the user in the btle_hci_adv_init() function is triggered when there are available reports in the queue. The user may then use the btle_hci_adv_report_get() function to pull reports from the queue. The reports contains total valid/invalid packets received and an HCI btle_evt_t that indicates what kind of event triggered this report. 

NOTE: The BLE spec does not define any scan report events, so a special event type was added for this: BTLE_VS_EVENT_NRF_SCAN_REQUEST_RECEIVED. It contains an address and address type of the scanner in addition to RSSI and channel.

### Available function calls:

Initialize the timeslot advertiser. The IRQ parameter decides which 
software interrupt to use for the generated events. For each report
the advertiser generates, the selected IRQ flag will be set, and the user
may get events from the queue using btle_hci_adv_evt_get().

__void btle_hci_adv_init(IRQn_Type btle_hci_adv_evt_irq);__
***

Function for getting pending tsa reports. It is recommended to repeat this 
function until it returns false, as there may be more than one pending report
in the queue.

__bool btle_hci_adv_report_get(nrf_report_t* report);__
***															

Softdevice event handler for the timeslot advertiser. Takes any timeslot event generated
by Softdevice function sd_evt_get(). 

__void btle_hci_adv_sd_evt_handler(uint32_t event);__
***															

Set the advertisement parameters for the timeslot advertiser. See hci/btle.h for details about
the struct parameter

__void btle_hci_adv_params_set(btle_cmd_param_le_write_advertising_parameters_t* adv_params);__
***

Enable or disable advertisement

__void btle_hci_adv_enable(btle_adv_mode_t adv_enable);__
***

Set the content of the timeslot advertiser advertisement packets. This does not include
any headers or address, as this is controlled internally. To set the advertisement
address, use the btle_hci_adv_params_set() function.

__void btle_hci_adv_data_set(btle_cmd_param_le_write_advertising_data_t* adv_data);__
***

Set the content of the timeslot advertiser scan response packet. This is the packet the
beacon will send to respond to external scan requests.
This does not include any headers or address, as this is controlled internally. 
To set the advertisement address, use the btle_hci_adv_params_set() function.

__void btle_hci_adv_scan_rsp_data_set(btle_cmd_param_le_write_scan_response_data_t* scan_rsp);__
***

Add a scanner to the device whitelist. If the whitelist filter is enabled, only
scanners on this whitelist will be accepted. 

The filter can be enabled with the btle_hci_adv_params_set() function above.

__void btle_hci_adv_whitelist_add(btle_cmd_param_le_add_device_to_whitelist_t* whitelist_device);__
***

Remove a scanner from the device whitelist. If the whitelist filter is enabled, only
scanners on this whitelist will be accepted. 

The filter can be enabled with the btle_hci_adv_params_set() function above.

__void btle_hci_adv_whitelist_remove(btle_cmd_param_le_remove_device_from_whitelist_t* whitelist_device);__
***

Remove all scanners from the device whitelist. If the whitelist filter is enabled, only
scanners on this whitelist will be accepted. 

The filter can be enabled or disabled with the btle_hci_adv_params_set() function above. 
Note that this function does not disable the filter functionality.
 
__void btle_hci_adv_whitelist_flush(void);__
***