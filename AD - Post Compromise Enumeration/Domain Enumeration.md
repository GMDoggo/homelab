# We've Compromised a User—Now What?

When you’ve compromised a user, several tools can help you efficiently enumerate their privileges and the environment:

- **BloodHound**
- **PlumHound**
- **ldapdomaindump**
- **PingCastle**

These tools allow you to gather critical information, such as user privileges, group memberships, and potential attack paths in an Active Directory environment. Below is a breakdown of each tool and its usage.

---

## ldapdomaindump

We previously used this tool in our IPv6 relay attack. If IPv6 is disabled, but we still manage to obtain an account, ldapdomaindump can be valuable for gathering information.

**Step 1:** Create a directory to hold the output and navigate into it.  
![Directory creation](../../Images/Pasted%20image%2020250220205343.png)

**Step 2:** Run ldapdomaindump with the user credentials:

```command
sudo /usr/bin/ldapdomaindump ldaps://192.168.163.5 -u 'QUIGLEY\j.smith' -p Password1
```

![](../../Images/Pasted%20image%2020250220210253.png)

**Step 3:** Analyze the output files.  
This tool provides insights into domain information like group memberships, privileges, and user relationships that can help you understand the environment.

---

## BloodHound

BloodHound is a powerful tool for enumerating Active Directory environments. It can help identify attack paths and privilege escalation opportunities.

**Step 1:** Start the Neo4j console.

`sudo neo4j console`

Neo4j is required to run BloodHound. This command starts the database that BloodHound uses to analyze Active Directory data.

**Step 2:** Check if Neo4j is running on the localhost:

`2025-02-21 02:04:25.864+0000 INFO  Bolt enabled on localhost:7687`

**Step 3:** Once Neo4j is running, you can start BloodHound.

`sudo bloodhound`

Once logged in, you'll find that the database is empty, and you’ll need to collect data for analysis using an ingestor.

**Step 4:** Set up the ingestor:

`sudo bloodhound-python -d quigley.local -u j.smith -p Password1 -ns 192.168.163.5 -c all`

- **-ns** specifies the name server (domain controller).
- **-c** specifies the data you want to collect (in this case, all data).

![](../../Images/Pasted%20image%2020250220211326.png)

**Step 5:** Import the collected JSON files into BloodHound using the GUI.  
This allows you to start analyzing the data visually.

![](../../Images/Pasted%20image%2020250220211522.png)

**Step 6:** Begin enumerating!  
Once the data is loaded, you can start exploring the environment, identifying attack paths, and looking for privilege escalation opportunities.

![](../../Images/Pasted%20image%2020250220211647.png)

---

## PlumHound

**IMPORTANT:** Make sure BloodHound is running, as PlumHound relies on BloodHound's Neo4j database for its analysis.

**Test Command:**

`sudo python3 PlumHound.py --easy -p neo4j1`

This test ensures PlumHound is pulling data from Neo4j properly.

**Generate Reports:**

`sudo python3 PlumHound.py -x tasks/default.tasks -p neo4j1`

This command generates default task reports that help analyze the environment for privilege escalation paths.

For more details, check out the GitHub repository: [PlumHound GitHub](https://github.com/PlumHound/PlumHound).

**Step 1:** Navigate to the reports directory:

`cd /reports`

**Step 2:** Open the report in a browser:

`firefox index.html`

By opening the report in a browser, you can visualize and analyze PlumHound's findings.

---

## PingCastle

While not covered in this guide, **PingCastle** is another tool that evaluates Active Directory security posture and helps identify potential misconfigurations, vulnerabilities, and attack paths.