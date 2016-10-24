#!/bin/bash
#Writen by Ilshat Akhmetzyanov 06/2016
#Ok I have no idea of where the process puts log or if it writes in stdout.
#So I assume that each process writes each own log file.
#And my script must work on each log file.
#Script should be started like: 				$./server_script.sh | nc $server_ip $server_port 
#(or with the start script)
#Several process ask to write different log files 
#(if I understood the task clear) and
#We should to start several scripts on different ports 
#On the client side the following command must be started 
#for each port i.e. for each process
#Scheme is (N servers, M process, N*M ports, 
#1 client with N*M listening ports, 1 client log file)
# 								$nc -l $server_port >> ./resulting_log_file.txt
#Feature is that in on the client side multiple NETCAT 
#process may write in one file wich we can watch with 
#								$watch tail ./resulting_log_file.txt
#when I tested script I used 2 PCs and at first started 
#netcat at a client and started watching a log file.
#Then I started server script at the server with piping to netcat
#then I manually added the following test strings block
#to a main process log file and save that file.
#Then I get information in a client in a watching console.

#defining of the month names
month="Jan Feb Mar Apr May June July Aug Sept Oct Nov Dec";
#loop variable
stop_var=0;
#initialize of variables
mass=0;
#here we chose the current process output log file

#log_file="./somelog.log";
#log_file_backup="./somelog_backup.log";

log_file="./test2.txt";
log_file_backup="./test2_backup.txt";

#this needed if the script starts first time
cp -f $log_file $log_file_backup;

#defining of log filesfile value
mass_new=$(du -b ./$log_file_backup | awk {'print $1'});
mass=$mass;

#main loop
while [[ $stop_var -ne 1 ]]
    do

    sleep 1;
    mass_new=$(du -b ./$log_file | awk {'print $1'});
    mass=$(du -b ./$log_file_backup | awk {'print $1'});

#grow of the log file definitely gives the reason to check on change
#then script looks for needed information and combinig the output string
    if [[ $mass_new -gt $mass ]]
    then
        compare_text=$(diff -n $log_file_backup $log_file);
        check_month=$(echo $compare_text | awk -F " " {'print $7'} | grep -oh "[[:alpha:]]*");
	for i in $month 
	do
	    if [[ "$i" = "$check_month" ]]
		then
		prepare_date_time=$(echo $compare_text | awk -F " " {'print $7 " " $9'});
		prepare_port=$(echo $compare_text | awk -F .port= {'print $2'} | awk -F -Dcom {'print $1'});
		prepare_hostname=$(echo $compare_text | awk -F .hostname= {'print $2'} | awk -F -Dcom {'print $1'});
		result_line="date=$prepare_date_time port=$prepare_port host=$prepare_hostname";
	    fi
	done
#after the script catched changes it moves all log info into backup to be able compare changes next time
	cp -f $log_file $log_file_backup;
#output sting
	echo $result_line;
    fi
done

