<h2 id="lora-port">Porting LoRaWAN stack and LoRa RF drivers</h2>

Mbed OS contains a native LoRaWAN stack (inside the Mbed OS tree) augmented with most commonly used LoRa RF drivers (outside the Mbed OS tree). Arm encourages developers around the globe to contribute to any of the Mbed OS features and functionality, including the LoRaWAN stack and related RF drivers using the [general guidelines provided](/docs/development/reference/guidelines.html).

The information in this section can be classified in two subsections:

- Porting a LoRa RF driver for the Arm Mbed LoRaWAN stack or any other LoRaWAN stack.
- Porting a third party LoRaWAN stack to Mbed OS .

The idea is to achieve universal application portability. In other words, a single application is compatible with the Arm Mbed LoRaWAN stack and any other third party LoRaWAN stack harnessing Arm Mbed OS.

The whole porting process consists of two key ingredients:

- An implementation of the [LoRaRadio](https://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_radio.html) class.
- An implementation of the [LoRaWANBase](https://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_w_a_n_base.html) class.

### Porting a LoRa RF driver

Arm Mbed OS provides a generic API that serves as a template for any LoRa RF driver. [LoRaRadio](https://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_radio.html) is a pure virtual class and is an attempt to standardize the APIs across all LoRa radios. Mbed Enabled LoRa radio driver implementations present as a LoRaRadio.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-os-docs-images/lora_radio_inherit.png)<span>Figure 1. Existing Mbed LoRa RF drivers inherit from the LoRaRadio class.</span></span>

For a reference implementation, please see the existing [LoRa RF drivers](https://github.com/ARMmbed/mbed-semtech-lora-rf-drivers). Construction of a LoRaRadio object is a matter of taste. The existing reference drivers allow construction of the LoRaRadio object with full pin definitions to make sure that the driver is usable across platforms with any pin combination. You are free to use any form of construction as long as you provide a LoRaRadio object down to the Arm Mbed LoRaWAN stack. Use of an instance of the `LoRaRadio` class for a third party LoRaWAN stack is beyond the scope of this documentation.

For API use cases, details, explanation and meaning, please see the `LoRaRadio` class reference below. We carefully planned and designed the data structures provided in [LoRaRadio.h](https://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/_lo_ra_radio_8h_source.html). They carry most of what you need to write your LoRa RF driver.

[![View code](https://www.mbed.com/embed/?type=library)](http://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_radio.html)

### Porting a third party LoRaWANStack

The vision driving Arm Mbed OS entails one operating system for myriad IoT technologies encompassing a multitude of devices or platforms. However, it does not limit the user to design something specific or tailored to his or her needs. We designed Arm Mbed LoRaWAN APIs in such a way that a developer can totally replace the native Mbed OS LoRaWAN stack with one of his or her own.

This subsection discusses how a third party LoRaWAN stack can seamlessly provide services to existing Mbed OS LoRaWAN applications or can reuse applications with minimal effort in terms of code.

<span class="notes">**Note:** The way a third party LoRaWAN stack harnesses the powers of Arm Mbed OS, in other words, synchronization methods (if using RTOS), timers, HAL and so on is beyond the scope of this documentation.</span>

The [LoRaWANBase](https://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_w_a_n_base.html) class is a pure virtual class providing user facing APIs for a LoRaWAN stack.

The native Arm Mbed LoRaWAN stack implements `LoRaWANBase` as [LoRaWANInterface](https://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_w_a_n_interface.html), which then serves as a network interface for the application. Potentially, any developer or vendor can provide an implementation of `LoRaWANBase`, and that particular implementation would serve as a network interface for the application.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-os-docs-images/lora_base.png)<span>Figure 2. Inheriting from LoRaWANBase to provide portable APIs.</span></span>

There can be many different scenarios when it comes to devices supporting LoRaWAN technology:

**Case 1: Design based on a LoRa chipset**

This design pattern follows the generic architecture with the LoRa radio part based on the LoRa transceiver and antenna plus associated RF circuitry from any vendor. The developer chooses only the LoRa radio chipset and the LoRaWAN stack, and the application runs on the MCU. In this case, you can use any Mbed compatible LoRaWAN stack, in other words, the native Mbed OS LoRaWAN stack or your ported LoRaWAN stack.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-os-docs-images/lora_radio_chipset.png)<span>Figure 3. Design based on LoRa RF chipset.</span></span>

**Case 2: Design based on a LoRa module**

A LoRa module means that the LoRa transceiver and an MCU are bound together (not in the same silicon package). This can be considered as a unit with all the connections between radio and MCU, plus antenna and other RF circuitry the vendor has already taken care of for the developer. For a developer, it means a board that contains a target MCU and RF transceiver integrated on the board (not on a chip). Arm Mbed OS provides the LoRaWAN stack and application, and the developer can modify the example application for his or her needs.

**Case 3: Design based on a LoRa RF-MCU**

An RF-MCU is an SoC including an MCU and LoRa transceiver on the same silicon package. From a developer’s point of view, this design is identical to a module-based design described in Case 2.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-os-docs-images/lora_module.png)<span>Figure 4. Design based on LoRa module (both as integrated on board and on chip).</span></span>

**Case 4: Design based on a LoRa modem**

A LoRa modem is a component that contains a stack and RF circuitry as a full package, mostly wired to a host MCU. In this case, if the developer wishes to be compliant with the existing applications, he or she may choose to write an adapter layer, which should be an implementation of `LoRaWANBase` and under the hood it could be AT commands to control the modem.

<span class="images">![](https://s3-us-west-2.amazonaws.com/mbed-os-docs-images/lora_modem.png)<span>Figure 5. Design based on LoRa modem.</span></span>

Please follow the detailed reference of `LoRaWANBase` to understand what these APIs and related data structures mean and why are they designed in this way.

You must implement the `initialize(events::EventQueue *queue)` API. Our design philosophy is that we wish to support the tiniest of devices with very little memory, and an event queue shared between the application and network stack is the best option in terms of memory.

[![View code](https://www.mbed.com/embed/?type=library)](http://os-doc-builder.test.mbed.com/docs/development/mbed-os-api-doxy/class_lo_ra_w_a_n_base.html)

### Testing

<span class="notes">**Note:** We'll publish automated tests for LoRa soon. For Mbed Partners that want to start porting Lora drivers, please contact your Partner lead.</span>
