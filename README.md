#CanadaSeismicDownloader

Welcome to the NRCAN Data Downloader (Canada Seismic Downloader) repository! This Python script allows you to easily download data from 
Natural Resources Canada (NRCAN), specifically designed for use with EQTransformer. In case you are struggling with networks like POLARIS that
their data is only available on NRCAN, you can benefit from this code.

## Installation

To get started with nrcandownloader, follow these simple steps:

1. **Clone the Repository**
    ```
    git clone https://github.com/sinamahani/CanadaSeismicDownloader.git
    ```

2. **Make the Script Executable**
    After cloning the repository, navigate to the project directory:
    ```
    cd CanadaSeismicDownloader
    ```
    Then make the main script executable by running:
    ```
    chmod +x wavedl
    ```

## Usage

To use the script, simply run:

```
./wavedl --network PO --station ARVN --begin 2005-07-02 --end 2005-08-02 --directory data
```

The script will prompt you for any necessary input such as the dataset name or specific parameters. Follow the 
on-screen instructions to download the desired data from NRCAN in a format suitable for EQTransformer.

Note that you'll need a proper internet connection and appropriate access permissions to NRCAN data resources.

## Help

For additional information, run:
```
./wavedl --help
```
This will display the list of available options and their descriptions. If you encounter any issues or have 
questions while using nrcandownloader, feel free to submit an issue here on GitHub or reach out via email at 
sina.sabermahani@gmail.com.

## Contributing

We appreciate your interest in contributing to CanadaSeismicDownloader! If you would like to submit changes or 
improvements, please fork this repository, make your modifications, and create a pull request.

## License

nrcandownloader is open-source under the MIT license. See [LICENSE](LICENSE) for more details.

If you have any questions or encounter issues while using nrcandownloader, feel free to submit an issue here on 
GitHub or reach out via email at yourproject@example.com.
