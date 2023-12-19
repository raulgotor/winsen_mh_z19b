<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Thanks again! Now go create something AMAZING! :D
***
***
***
*** To avoid retyping too much info. Do a search and replace for the following:
*** raulgotor, winsen_mh_z19b, twitter_handle, Winsen MH-Z19B Driver, Platform agnostic C driver for the Winsen MH-Z19B Infrared CO2 Sensor
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]

<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://github.com/raulgotor/winsen_mh_z19b">
  </a>
</p>

<h3 align="center">Winsen MH-Z19B Driver</h3>

  <p align="center">
    Platform agnostic C driver for the Winsen MH-Z19B Infrared CO2 Sensor
    <br />
    <a href="https://github.com/raulgotor/winsen_mh_z19b"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/raulgotor/winsen_mh_z19b">View Demo</a>
    ·
    <a href="https://github.com/raulgotor/winsen_mh_z19b/issues">Report Bug</a>
    ·
    <a href="https://github.com/raulgotor/winsen_mh_z19b/issues">Request Feature</a>
  </p>

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
  <ol>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT

[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fraulgotor%2Fwinsen_mh_z19b.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fraulgotor%2Fwinsen_mh_z19b?ref=badge_large)

## About The Project

[![Product Name Screen Shot][product-screenshot]](iu.jpg)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fraulgotor%2Fwinsen_mh_z19b.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fraulgotor%2Fwinsen_mh_z19b?ref=badge_shield)

Here's a blank template to get started:
**To avoid retyping too much info. Do a search and replace with your text editor for the following:**
`raulgotor`, `winsen_mh_z19b`, `twitter_handle`, `Winsen MH-Z19B Driver`, `Platform agnostic C driver for the Winsen MH-Z19B Infrared CO2 Sensor`


### Built With

* []()
* []()
* []()

-->

<!-- GETTING STARTED -->
## Getting Started

To get a local copy up and running follow these simple steps.

### Installation

1. Navigate to your project's source directory

2. Clone the repo
   ```sh
   git clone https://github.com/raulgotor/winsen_mh_z19b.git
   ```
3. Write a transfer function (see next section)


<!-- USAGE EXAMPLES -->
## Usage

### Transfer function

A transfer function glues the driver logic with the current device specific API.
The transfer function needs to be injected to the driver, so it can access your 
device specific UART peripheral while remaining agnostic to the platform you are
using it at.

The transfer function should match with the following prototype:

  ```c 
  /*!
   * @brief UART transfer function prototype
   *
   * The transfer function that will be registered with the `mh_z19b_init` function
   * must follow this prototype
   *
   * @note If the operation to perform is a read operation:
   *          - `p_tx_buffer` must be null
   *          - `tx_buffer_size` must be 0
   *
   * @note If the operation to perform is a write operation:
   *          - `p_rx_buffer` must be null
   *          - `rx_buffer_size` must be 0
   *
   * @note `p_rx_buffer` and `p_tx_buffer` cannot be both null
   *
   * @param[out]          p_rx_buffer         Pointer to the buffer where to read
   *                                          the data to
   * @param[in]           rx_buffer_size      Size of the read buffer
   * @param[in]           p_tx_buffer         Pointer to the buffer where to read
   *                                          the data from
   * @param[in]           tx_buffer_size      Size of the write buffer
   *
   * @return              mh_z19b_error_t      Operation result
   * @retval              MH_Z19B_ERROR_SUCCESS
   *                                          Everything went well
   * @retval              MH_Z19B_ERROR_BAD_PARAMETER
   *                                          Parameter is null
   */
  typedef mh_z19b_error_t (*mh_z19b_xfer_func )(uint8_t * const p_rx_buffer,
                                              size_t const rx_buffer_size,
                                              uint8_t const * const p_tx_buffer,
                                              size_t const tx_buffer_size);
     
   ```

#### Transfer function and module initialization example
An example of a transfer function for Espressif's esp-idf would be:

```c
static mh_z19b_error_t xfer_func(uint8_t const * const p_rx_buffer,
                                 size_t const rx_buffer_size,
                                 uint8_t * const p_tx_buffer,
                                 size_t const tx_buffer_size)
{

        mh_z19b_error_t result = MH_Z19_ERROR_SUCCESS;
        bool is_rx_operation = true;
        int uart_result;

        if ((NULL == p_rx_buffer) != (0 == rx_buffer_size)) {
                result = MH_Z19B_ERROR_BAD_PARAMETER;

        } else if ((NULL == p_tx_buffer) != (0 == tx_buffer_size)) {
                result = MH_Z19B_ERROR_BAD_PARAMETER;

        } else if ((NULL == p_rx_buffer) && (NULL == p_tx_buffer)) {
                result = MH_Z19B_ERROR_BAD_PARAMETER;
        } else if (0 != tx_buffer_size) {
                is_rx_operation = false;
        }

        if ((MH_Z19B_ERROR_SUCCESS == result) && (is_rx_operation)) {
                uart_result = uart_read_bytes(UART_NUM_2,
                                              (void *)p_rx_buffer,
                                              (uint32_t)rx_buffer_size, 100);

                if (-1 == uart_result) {
                        result = MH_Z19B_ERROR_IO_ERROR;
                }

        } else if (MH_Z19B_ERROR_SUCCESS == result) {
                uart_result = uart_write_bytes(UART_NUM_2,
                                               (void *)p_tx_buffer,
                                               (uint32_t)tx_buffer_size);

                if (-1 == uart_result) {
                        result = MH_Z19B_ERROR_IO_ERROR;
                }
        }

        return result;
}
  ```

Below an implementation example for an ESP32 MCU using esp-idf is suggested:

- Device specific UART bus configuration and initialization
- Driver initialization with injection of the transfer function

1) Initialize the UART bus
   ```c

   static uart_port_t const m_uart_instance = UART_NUM_2;
   static int const m_uart_tx_pin = 33;
   static int const m_uart_rx_pin = 32;

   bool sensor_init(void) {

        uart_config_t const uart_config = {
                        .baud_rate = 9600,
                        .data_bits = UART_DATA_8_BITS,
                        .parity = UART_PARITY_DISABLE,
                        .stop_bits = UART_STOP_BITS_1,
                        .flow_ctrl = UART_HW_FLOWCTRL_DISABLE,
        };

        int const uart_buffer_size = (1024 * 2);
        bool success = true;

        esp_err_t esp_result;
        QueueHandle_t uart_queue;
        BaseType_t task_result;
        mh_z19b_error_t mh_z19_result;

        esp_result = uart_set_pin(
                        m_uart_instance,
                        m_uart_tx_pin,
                        m_uart_rx_pin,
                        UART_PIN_NO_CHANGE,
                        UART_PIN_NO_CHANGE);

        success = (ESP_OK == esp_result);

        if (success) {
                esp_result = uart_param_config(m_uart_instance, &uart_config);

                success = (ESP_OK == esp_result);
        }

        if (success) {
                esp_result = uart_driver_install(
                                m_uart_instance,
                                uart_buffer_size,
                                uart_buffer_size,
                                10,
                                &uart_queue,
                                0);

                success = (ESP_OK == esp_result);
        }
        
        if (success) {
                mh_z19b_result = mh_z19b_init(xfer_func);

                success = (MH_Z19B_ERROR_SUCCESS == mh_z19_result);
        }

        if (success) {
                mh_z19b_result = mh_z19b_enable_abc(false);

                success = (MH_Z19B_ERROR_SUCCESS == mh_z19_result);
        }
   }
   ```
  
### Reading the CO2 concentration

Below there is an example for performing temperature readings.

```c
uint32_t co2_ppm;
mh_z19b_error_t result;

result = mh_z19b_get_gas_concentration(&co2_ppm);


if (MH_Z19B_ERROR_SUCCESS == result) {
        printf("CO2 concentration is %d ppm\n", co2_ppm);
}
```

### Further documentation

_Please refer to the in code documentation and to the [Winsen MH-Z19B Datasheet](https://www.winsen-sensor.com/d/files/MH-Z19B.pdf)_



<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/raulgotor/winsen_mh_z19b/issues) for a list of proposed features (and known issues).



<!-- CONTRIBUTING -->
## Contributing

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.



<!-- CONTACT -->
## Contact

Raúl Gotor

Project Link: [https://github.com/raulgotor/winsen_mh_z19b](https://github.com/raulgotor/winsen_mh_z19b)


<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

* [Embedded C Coding Standard, 2018 Michael Barr](https://barrgroup.com/sites/default/files/barr_c_coding_standard_2018.pdf)
* [Best README template](https://github.com/othneildrew/Best-README-Template)


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/raulgotor/winsen_mh_z19b.svg?style=for-the-badge
[contributors-url]: https://github.com/raulgotor/winsen_mh_z19b/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/raulgotor/winsen_mh_z19b.svg?style=for-the-badge
[forks-url]: https://github.com/raulgotor/winsen_mh_z19b/network/members
[stars-shield]: https://img.shields.io/github/stars/raulgotor/winsen_mh_z19b.svg?style=for-the-badge
[stars-url]: https://github.com/raulgotor/winsen_mh_z19b/stargazers
[issues-shield]: https://img.shields.io/github/issues/raulgotor/winsen_mh_z19b.svg?style=for-the-badge
[issues-url]: https://github.com/raulgotor/winsen_mh_z19b/issues
[license-shield]: https://img.shields.io/github/license/raulgotor/winsen_mh_z19b.svg?style=for-the-badge
[license-url]: https://github.com/raulgotor/winsen_mh_z19b/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/raulgotor