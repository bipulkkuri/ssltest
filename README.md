### SSL Testing and ansible

- There are scenarios where OPS team/security team would like to test your SSL strength ,algorithm /configurations/vulnerabilities
- https://testssl.sh/ provides a compact shell script with docker file too. Please donate to  them for the awesome work they are doing
- If you have docker configured on your box a simple run will generate a proper report
`docker run -ti drwetter/testssl.sh <<TARGET_IPADDRESS>>`
- testssl.sh script has lot of options just do
`testssl.sh --help`
- My use case was to do SSL testing for 500+ servers ofcourse I could loop through in shell but I tried to do it in ansible with local_action.Since I don't need to login to server openssl_certificate module is not going to help me .Below is a sample ansible yml file. testssml.yml


```
---
- hosts: localhost
  gather_facts: false
  tasks:
  - name: checkout testssl locally
    run_once: true
    git:
      repo: https://github.com/drwetter/testssl.sh.git
      clone: yes
      dest: "testssl.sh"
      depth: 1
      force: yes
      accept_hostkey: yes
    delegate_to: localhost

  - name: change permissions
    local_action: command chmod +x testssl.sh/testssl.sh

- hosts: localhost
  tasks:
    - name: run testssl on all servers
      local_action: shell {{playbook_dir}}/testssl.sh/testssl.sh {{ myOptions }} {{ item }}
      with_items: "{{ groups['all'] }}"
 ```
and the host file as
```
[all]
172.217.10.14
```


to run just run the ansible-playbook
```ansible-playbook -i hosts testssl.yml -e "myOptions=--html"```
This is a actual output of the program in a html file.The script will create HTML to file '${NODE}-p${port}${YYYYMMDD-HHMM}.html' file in the same folder where it was executed. Use myOptions var to pass other commands.

![Report](https://miro.medium.com/max/3122/1*aGBeRttvnfnDWcOVJCq1GA.png "SSL TEST REPORT")

##NOTES:
further testing /enhancement ,Check if root is allowed from ecternal locations
ssh -o "StrictHostKeyChecking no" -o "ConnectTimeout=10"  -T root@<<TARGET_IP>>
```
$ssh -o "StrictHostKeyChecking no" -o "ConnectTimeout=10"  -T root@172.217.10.14
ssh: connect to host 172.217.10.14 port 22: Connection timed out
```
