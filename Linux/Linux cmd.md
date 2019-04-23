#### Linux Command

##### Firewall

firewall-cmd --state    //Check Firewall state

systemctl start firewalld.service //Start Firewall,Make Firewall Running

firewall-cmd --zone=public --add-port=8080/tcp --permanent  //Open port.8080 just a example

systemctl restart firewalld.service  //Restart Firewall

firewall-cmd --reload  // Reload Firewall Config

firewall-cmd --list-ports  //Check  Firewall Opening Port

systemctl stop firewalld.service   // Stop Firewall Running

systemctl disable firewalld.service // Disable Firewall Running in  System starting up

##### System

tar -zvxf  filename   //  Unzip

rz   // Upload File to the Linux Local 

rm   filename // Delete File

rm -rf   //Delete this  directory  all file

mv filename1  filename2  // rename   filename1 change  to filename2

