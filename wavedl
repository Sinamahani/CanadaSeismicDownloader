#!/usr/bin/env python3

import subprocess
import sys

try:
    import obspy  # Example package
except ImportError:
    print("Installing missing dependencies...")
    subprocess.check_call([sys.executable, "-m", "pip", "install", "-r", "requirements.txt"])


import obspy
from obspy.clients.fdsn import Client
import os
from urllib.error import HTTPError
import time
import argparse



# Create Argument Parser
parser = argparse.ArgumentParser(description='Download seismic data from NRCan specifically and IRIS FDSN web services in general.')

# Add Optional Arguments with Flags
parser.add_argument('-dir', '--directory', type=str, default="data", help='Directory to save the data (default: "data")')
parser.add_argument('-st', '--station', type=str, required=True, help='Station code, e.g. SNKN')
parser.add_argument('-nt', '--network', type=str, required=True, help='Network code, e.g. PO')
parser.add_argument('-bg', '--begin', type=str, default='1900-01-01', help='Start date in "YYYY-MM-DD" format')
parser.add_argument('-ed', '--end', type=str, default='2030-01-01', help='End date in "YYYY-MM-DD" format')
parser.add_argument('-cs', '--chunk_size', type=int, default=1, help='Number of days to download at a time')

# Parse Arguments
args = parser.parse_args()

# Assign Variables
directory = args.directory
st_code = args.station.upper()
nt_code = args.network.upper()
bg = obspy.UTCDateTime(args.begin)
ed = obspy.UTCDateTime(args.end)
chunk_size = args.chunk_size

print(f"-- Download Details:\n    Directory: {directory}\n    Station: {st_code}\n    Network: {nt_code}\n    Start Date: {bg}\n    End Date: {ed}\n    Chunk Size: {chunk_size}")

def _time_adjustment_and_sta_file(nt_code, st_code, bg, ed):
    client = Client("IRIS")
    
    try:
        os.mkdir('staXML')
    except FileExistsError:
        pass

    st_data = client.get_stations(network=nt_code, station=st_code, level='response')
    st_data.write(f"staXML/{nt_code}.{st_code}.xml", format='STATIONXML')
    ch_list = list(set([ch.code for ch in st_data[0][0].channels]))

    print(f"\nWorking on Station {st_code} ...")
    print(f"    {st_code} ========  located at {st_data[0][0].latitude}°N, {st_data[0][0].longitude}°E, {st_data[0][0].elevation}m above sea level.")
    #check if bg and ed are available and within the data range
    if bg > obspy.UTCDateTime(st_data[0][0].end_date) or ed < obspy.UTCDateTime(st_data[0][0].start_date):
        print(f"!!! Data not available for the requested time period !!!")
        return None, None, st_data, ch_list
    if bg < obspy.UTCDateTime(st_data[0][0].start_date):
        bg = obspy.UTCDateTime(st_data[0][0].start_date)
    if ed > obspy.UTCDateTime(st_data[0][0].end_date):
        ed = obspy.UTCDateTime(st_data[0][0].end_date)
    print(f"    Data available from {bg} to {ed}")
    return bg, ed, st_data, ch_list


def download_from_nrcan(nt_code: str, st_code: str, bg: str = None, ed: str = None, chunk_size: int = 1, directory: str = "data", **kwargs):
    '''
    Download seismic data from NRCan for a given station and network code.

    Parameters:
    nt_code (str): Network code, e.g. 'PO' for POLARIS network
    st_code (str): Station code, e.g. 'SNKN' for Snook's Arm station
    bg (str): Start date in 'YYYY-MM-DD' format
    ed (str): End date in 'YYYY-MM-DD' format
    chunk_size (int): Number of days to download at a time
    directory (str): Directory to save the data

    optional:
    sation_directory (str): Directory to save the station XML files

    Returns:
    None - saves the data in the specified directory
    '''

    chunk_size = chunk_size * 86400
    client = Client("IRIS")

    
    st_code = st_code
    nt_code = nt_code
    bg, ed, st_data, ch_list = _time_adjustment_and_sta_file(nt_code, st_code, bg, ed)
    print(f" The adjusted time period is from {bg} to {ed}")


    if bg and ed:

        while bg < ed:
            print(f"      Downloading data from {bg} to {bg + chunk_size}")
            date = bg.strftime("%Y%m%d")
            folder = "/".join([directory, st_code])
            
            try:
                os.makedirs(folder)
            except FileExistsError:
                pass
          
            bg_str = bg.strftime("%Y-%m-%d")
            bg_plus_chunk = (bg + chunk_size).strftime("%Y-%m-%d")
            url = f"https://www.earthquakescanada.nrcan.gc.ca/fdsnws/dataselect/1/query?starttime={bg_str}&endtime={bg_plus_chunk}&network={nt_code}&station={st_code}&nodata=404"
            
            try:
                wv = obspy.read(url)
            except:
                try:
                    wv = client.get_waveforms(nt_code, st_code, "*", "*", bg, bg + chunk_size)
                except:
                    wv = obspy.Stream()

            if len(wv) >= 1:
                print(f"        {date} ========  {len(wv)} traces downloaded.")

            for ch in ch_list:
                print(f"            Working on channel {ch}")
                wv_ch = wv.select(channel=ch)
                if len(wv_ch) >= 1:
                    wv_ch.detrend('demean')
                    wv_ch.detrend('linear')
                    wv_ch.merge(fill_value='interpolate')
                    # wv_ch.resample(100)
                    wv_ch.remove_response(inventory=st_data, output="VEL", pre_filt=(0.01, 0.02, 45, 50))
                    wv_ch.trim(starttime=bg, endtime=bg + chunk_size)
                    file_name = f"{wv_ch[0].stats.network}.{wv_ch[0].stats.station}.{wv_ch[0].stats.location}.{wv_ch[0].stats.channel}__{wv_ch[0].stats.starttime.strftime('%Y%m%d%H%M%S')}__{wv_ch[0].stats.endtime.strftime('%Y%m%d%H%M%S')}.mseed"
                    wv_ch.write(f"{folder}/{file_name}", format='MSEED')
                    print(f"                \u2713 \u2713{file_name} saved \u2713 \u2713")
                else:
                    print(f"                \u00D7 \u00D7 channel NOT found \u00D7 \u00D7")
            bg += chunk_size
        print(f"Data downloaded successfully.")
    else:
        print(f"The program terminated. Please check your input parameters.")





if __name__ == "__main__":
    download_from_nrcan(nt_code, st_code, bg=bg, ed=ed, chunk_size=1, directory=directory)
