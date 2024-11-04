# AutoDAQ
AutoDAQ is a set of linux terminal commands meant to be used in conjunction with a multichannel CAEN digitizer PMT setup. It is an an add-on to the wavedump code to perform data analysis specifically for Lucas Cell based scintillation measurement as a one-liner command. 


## Installation 

1. ssh into science2 using the following command :
```
ssh <USERNAME>@science2.snolab.ca
```
2. In your directory, execute the following command and exit ssh connection
```
mkdir wavedumpoutput
exit
```
3. Copy and paste the following code at the end of the /home/[username]/.bashrc file
```
function getLC() {
    # Input parameters
    local channel=$1 # Channel number
    local year=$2 # Year(YYYY) of wavedump file
    local month=$3 # Month(MM) of wavedump file
    local day=$4 # Day(DD) of wavedump file
    local order=$5 # Order of the most recent file (1 being most recent)

    # User credentials and directories
    local target_user="<USERNAME>" # Username for science2
    local target_password="<PASSWORD>"  # Password for science2
    local local_dir="<LOCAL_PATH>" # Path on local machine

    # Remote server credentials and directories
    local remote_user="nasim_admin" # Remote server username
    local remote_host="lbc.internal.snolab.ca" # Remote server hostname
    local remote_dir="wavedumpoutput" # Path on remote server
    local target_host="science2.snolab.ca" # Science2 hostname
    local target_dir="/users/${target_user}/wavedumpoutput"  # Path on science2
    local analysis_executable="/users/nasim/RadonCountCode/AlphaCounting_v3.exe"  # Counting executable path on science2
    local wave_dump_output_dir="/users/${target_user}/wavedumpoutput"  # Path on science2
    local remote_password="Radon226"  # Password for lbc.internal.snolab.ca
    
    # Check if day, month, and year are provided
    if [[ -n $day && -n $month && -n $year ]]; then
        echo "Searching for files on $year/$month/$day"
        local recent_file=$(sshpass -p "$remote_password" ssh "$remote_user@$remote_host" "cd $remote_dir && ls -t wave$channel$year-$month-$day*.txt | head -n 1")
    # Check if order is provided
    elif [[ -n $order ]]; then
        echo "Searching for $order most recent files"
        local recent_file=$(sshpass -p "$remote_password" ssh "$remote_user@$remote_host" "cd $remote_dir && ls -t wave$channel$year-$month-$day*.txt | head -n $order")
    else
        echo "Searching for files without date filter"
        local recent_file=$(sshpass -p "$remote_password" ssh "$remote_user@$remote_host" "cd $remote_dir && ls -t wave$channel*.txt | head -n 1")
        # Existing logic to search files
    fi

    # Use sshpass to SSH into the remote host and find the most recent file for the given channel
	echo "Recent File : "$recent_file
    # Copy the most recent file from the remote host to the local directory
    sshpass -p "$remote_password" scp "$remote_user@$remote_host:$remote_dir/$recent_file" "$local_dir"
	echo "Copied File(lbc.internal->local) : "$recent_file" to "$local_dir
    # Copy the file from the local directory to the target remote directory on science2
    sshpass -p "$target_password" scp "$local_dir/$recent_file" "$target_user@$target_host:$target_dir"
	echo "Copied File(local->science2) : "$recent_file" to "$target_dir 
    # Execute the C++ analysis executable on the copied file on science2
    sshpass -p "$target_password" ssh "$target_user@$target_host" "source /usr/local/root_versions/root-6.08.06/bin/thisroot.sh;$analysis_executable $target_dir/$recent_file $wave_dump_output_dir; exit"
	echo "Analyzed File : "$recent_file
}
```

4. Replace [USERNAME], [PASSWORD] with your science2 login credentials and <LOCAL_PATH> with your preferred intermediate directory. This intermediate directory serves to circumvent troublesome authorization issues that arise from interhost direct ssh file transfer.
5. Save the /home/[username]/.bashrc file
6. Exit WSL and start a new instance
8. Try the examples below to ensure everything was properly installed

Examples
```
getLC 0
```
This obtains the most recent wavedump file from channel 0.

```
getLC 5 2022 10 11 1
```
This obtains the last wavedump file created on the 11th of October 2022 from channel 5. 

## Usage

The command is used with the following syntax : 
```
getLC <CHANNEL> <YEAR(YYYY)> <MONTH(MM)> <DAY(DD)> <ORDER>
```
CHANNEL : PMT channel from which the data was collected (Required)

YEAR(YYYY) : The year during which the data was collected in 4 digit format (If date(year/month/day) is left blank the most recent file from the selected channel will be chosen)

MONTH(MM) : The month during which the data was collected in 2 digit format (If date(year/month/day) is left blank the most recent file from the selected channel will be chosen)

DAY(DD) : The day during which the data was collected in 2 digit format (If date(year/month/day) is left blank the most recent file from the selected channel will be chosen)

ORDER : The order of the desired file if more than one were created in one day. (1 = Most Recent, 2=second most recent... If left blank, the most recent one will be chosen)
