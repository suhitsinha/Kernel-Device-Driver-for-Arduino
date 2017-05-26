1. Goal of the project
We proposed to write an USB driver for Arduino board at the project proposal. The goal of our project is to build a custom USB driver that can communicate with the Arduino Board. Managing USB devices are an important part for any operating system nowadays. Current devices such as flash drives, routers, cameras etc. nowadays communicates through USB connections only. It also becomes a custom to use the USB
port for the ease of uniformity and simplicity. As part of this course project we tried to build and understand the process of the USB communication stating from plugin the device to how it communicate using the hardware until the point it gets disconnected. In order to accomplish the objective, we choose an USB Arduino device and tried to build a USB device driver for sending some data to the same. We chose Arduino in order to show the validation visually as the Arduino board holds the capability of displaying any message to the LCD display connected to it.


2. Project Description
In a summarized manner we could put the project tasks into perspective as follows:
a. The driver will capture the data entered by user on a write request to the file and will send it to the Arduino device.
b. The process of sending the data needs to be handled properly with proper control URB packets and write URB packets.
c. The data. If received successfully by Arduino, will be displayed in the LCD display connected to the Arduino board.
We will discuss each of these steps in details in the next section. However, to state it briefly, this complete set of task needs handling the user data, probing and set up the Arduino device properly when connected, creating the URB packets for both control and bulk transfer and until the device got disconnected.


3. Details of Arduino USB connector:

The Arduino USB communicates through the following end points:
 Control End point - For sending a control packet to the USB line
 Bulk out End point - For transmitting data
 Bulk in End point - For receiving data
We used those end point send the data and control message to the Arduino board in order to communicate properly.


4. Technical Requirements
As mentioned earlier, one of the primary reasons we have chosen Arduino was to make the validations more easy and appealing. Arduino board has an inbuilt functionality of displaying some message to the LCD 16X2 display. We used this feature for our validation purpose. One of the other assumptions we made at the beginning of the project is that, due to the availability of the extensive Arduino documentation, we will find it much easier to find the lines and endpoints of the USB connection. But during our approach to the project building we found out that, the documentation available for Arduino are available for their software development kit only. It had not given us enough insight about the USB communication data point. Hence, we have to perform some reverse engineering to know the whereabouts regarding the USB transfer details to Arduino. It’s described in the next section.


5. Reverse Engineering
As mentioned earlier, that we performed some reverse engineering in order to find the details of the control parameters such as line encoding values and baud rate of the connection. The baud rate should be same as the baud rate set in the Arduino board to receive the data. Also, the reverse engineering helped us to found out the control end point of the board which is needed to send the control data. We used “wireshark” tool to sniff the USB port in which we connected the Arduino board.


6. Probing the device
This functionality will be invoked when a specific device was plugged in to the host USB port. At the beginning of out kernel module, we registered a USB device for the kernel module with the vendor id and product id. The Arduino board details are as follows:
Vendor Id: 0x2341 and Product Id: 0x0043
This is a unique identifier for any USB device. Whenever a USB device with the above specified identifiers are plugged into the host, the probe function corresponding to theArduino device will be invoked. In the probe function, the following steps have been performed:
a. We first find the Bulk in and Bulk out endpoint corresponding to the Arduino board using the following code snippet:

	for (i = 0; i < arduino_currsetting->desc.bNumEndpoints; ++i) {
		endpoint = &arduino_currsetting->endpoint[i].desc;
		if (((endpoint->bEndpointAddress & USB_ENDPOINT_DIR_MASK) == USB_DIR_IN) && ((endpoint->bmAttributes & USB_ENDPOINT_XFERTYPE_MASK) == USB_ENDPOINT_XFER_BULK))
		{
			dev->bulk_in_endpoint = endpoint;
			printk("Found Bulk Endpoint IN\n");
		}
		if (((endpoint->bEndpointAddress & USB_ENDPOINT_DIR_MASK) == USB_DIR_OUT) && ((endpoint->bmAttributes & USB_ENDPOINT_XFERTYPE_MASK) == USB_ENDPOINT_XFER_BULK))
		{
			dev->bulk_out_endpoint = endpoint;
			printk("Found Bulk Endpoint OUT\n");
		}
	}
b. Then we create the control packet to send the line encoding and baud rate of the USB line.
c. Register the plugged Arduino board usb_register_dev function.


7. Send the data to the device
The overridden write system call creates the require URB packets and send the data through the USB line to Arduino board. The functionalities of write function has been described below:
a. Copy the data from the user process to the buffer of the module.
b. Creates a URB packet and allocate buffer it.
c. Fill the URB packet using the function usb_fill_bulk_urb. The usb_fill_bulk_urb functions takes as parameter:
	[ struct urb * urb, struct usb_device * dev, unsigned int pipe, void * transfer_buffer, int buffer_length, usb_complete_t complete_fn, void * context ]
d. Submit the URB packet using usb_submit_urb call to the given end point.
