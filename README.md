# AutoDAQ
AutoDAQ is a set of linux terminal commands meant to be used in conjunction with a multichannel CAEN digitizer PMT setup. It is an an add-on to the wavedump code to perform data analysis specifically for Lucas Cell based scintillation measurement as a one-liner command. 


## Installation 

1.Copy and paste the following code at the end of the /home/[username]/.bashrc file

```
function getLC() {
    local channel=$1
    local remote_user="nasim_admin"
    local remote_host="lbc.internal.snolab.ca"
    local remote_dir="wavedumpoutput"
    local local_dir="/your/path/here/wavedumpoutput"
    local analysis_executable="/your/path/here/RadonCountCode/AlphaCounting_v3.exe"
    local wave_dump_output_dir="/your/path/here/wavedumpoutput"
    local password="password"
	
	
    # Use sshpass to SSH into the remote host and find the most recent file for the given channel
    local recent_file=$(sshpass -p "$password" ssh "$remote_user@$remote_host" "cd $remote_dir && ls -t wave$channel*.txt | head -n 1")
	
    # Copy the most recent file to the local directory
    sshpass -p "$password" scp "$remote_user@$remote_host:$remote_dir/$recent_file" "$local_dir"
	
    echo "From : "$remote_user@$remote_host
    echo "You are currently analyzing : "$recent_file
    # Execute the C++ analysis executable on the copied file
    "$analysis_executable" "$local_dir/$recent_file" "$wave_dump_output_dir"
}
```

2. Replace local_dir and wave_dump_output_dir with your local wavedumpoutput path, this is where wavedump files from the DAQ will be created and where wavedump files will be pulled from for analysis.
3. Replace analysis_executable with the path where the AlphaCounting_v3.exe file is stored. This file takes care of the analysis of the PMT data
4. Change the password with the nasim_admin@lbc.internal.snolab.ca password
5. Confirm that it works by executing :

```
getLC 0
```
This obtains the most recent wavedump file from channel 0.

