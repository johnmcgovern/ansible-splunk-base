# Ansible-Splunk-Base ./files/

By default, this project pulls down Splunk Enterprise and Universal Forwarder versions by wgetting them from the official Splunk download links.

If you would like to instead upload the Splunk .tgz from your local ansible machine, you can do so by modifying the download_tgz_from_splunk_servers and download_uf_tgz_from_splunk_servers (specifically setting either of them to "false") in ./group_vars/all

If a file with the same MD5 checksum is already present on the server, this project does not bother to redownload/upload the file.