# Amass

## Overview

Amass is a comprehensive **footprinting** and **fingerprinting** tool backed by the OWASP Foundation, designed for enumerating subdomains, Autonomous System Numbers (ASNs) and other assets owned by an organisation. It leverages various techniques such as DNS queries, web scraping, and API integrations to gather extensive information about a target domain.

Amass integrates with numerous data sources to enhance its footprinting capabilities, including:

AlienVault, ArchiveIt, ArchiveToday, Arquivo, Ask, Baidu, BinaryEdge, Bing, BufferOver, Censys, CertSpotter, CIRCL, CommonCrawl, Crtsh, [...], ViewDNS, VirusTotal, Wayback, WhoisXML, Yahoo - see full list [here](https://github.com/OWASP/Amass)

## Installation

Amass can be installed using the following methods:

1. **Package Managers**:
    Amass is available via package managers.

2. **Using Go**:
    If you have Go installed, you can use the following command:
    ```bash
    export GO111MODULE=on
    go get -v -u github.com/OWASP/Amass/v3/...
    cd $GOPATH/src/github.com/OWASP/Amass
    go install ./...
   ```

3. **Using Docker**:
    If you Docker usage is preferred, there is a Docker image available:
    ```bash
    docker build -t amass https://github.com/OWASP/Amass.git
    docker run -v OUTPUT_DIR_PATH:/.config/amass/ amass enum --list          # -v volume mapping for config and output
    ```

## Usage

### Amass Intel

The Amass intel subcommand can aid with collecting open source intelligence on the organisation and allow you to find further root domain names associated with the organisation.

To gather intelligence an organisation (example: owasp.org) and find associated root domains, use the following command:

```bash
$ amass intel -d owasp.org -whois

appseceu.com
owasp.com
appsecasiapac.com
appsecnorthamerica.com
appsecus.com
[...]
owasp.org
appsecapac.com
appsecla.org
[...]
```

ASNs are unique numbers assigned to Internet Service Providers (ISPs) and organizations that own or manage a network.
There is the possibility of enumerating organisational names with Amass which could return ASN IDs assigned to the target, an example is shown below:

```bash
$ amass intel -org 'Example Ltd'

111111, MAIN_PRODUCT -- Example Ltd
222222, SECONDARY_PRODUCT - Example Ltd
[...]

$ amass intel -active -asn 222222 -ip
some-example-ltd-domain.com 123.456.78.90
[...]
```

### Amass Enum

Amass is also capable of enumerating subdomains for a target domain using both passive and active techniques.

```bash
$ amass enum -passive -d owasp.org -src

[...]
[ThreatCrowd]     update-wiki.owasp.org
[...]
BufferOver]      my.owasp.org
[Crtsh]           www.lists.owasp.org
[Crtsh]           www.ocms.owasp.org
[...]
Querying VirusTotal for owasp.org subdomains
Querying Yahoo for owasp.org subdomains
[...]
```

The following commands uses the *-active* flag which enables zone transfers and port scanning of SSL/TLS services and grabbing their certificates to extract any subdomains from certificate fields (e.g. Common Name).
It instructs Amass to conduct active enumeration of the domain owasp.org using a brute-force approach with a specified wordlist located at /root/dns_lists/deepmagic.com-top50kprefixes.txt. It includes source information in the output and retrieves associated IP addresses while saving the output in a directory named amass4owasp. The command utilizes a custom configuration file located in /root/amass/config.ini and directs the results to amass_results_owasp.txt.

```bash
amass enum -active -d owasp.org -brute -w /root/dns_lists/deepmagic.com-top50kprefixes.txt -src -ip -dir amass4owasp -config /root/amass/config.ini -o amass_results_owasp.txt
```

![Amass Enumeration](../pics/amass1.png)

Let's assume that for some reason the OWASP organisation tends to create subdomains with "zzz" prefixes, such as zzz-dev.owasp.org. Amass' hashcat-style wordlist mask feature can be leveraged to brute-force all the combinations of "zzz-[a-z][a-z][a-z].owasp.org" using the following command:

```bash
amass enum -d owasp.org -norecursive -noalts -wm "zzz-?l?l?l" -dir amass4owasp
```

![Amass Mask Brute-Forcing](../pics/amass2.png)

### Amass Track

Amass track helps compare results across enumerations performed against the same target and domains. An example is the below command which compares the last 2 enumerations performed against "owasp.org". This is done by specifying the same Amass output folder and database been usied throughout these examples. The most interesting lines are the ones starting with the "Found" keyword and this means that the subdomain was not identified in previous enumerations.

```bash
amass track  -config /root/amass/config.ini -dir amass4owasp -d owasp.org -last 2
```

![Amass Track](../pics/amass3.png)

### Amass Db

This subcommand is used in order to interact with an Amass graph database, either the default or the one specified with the "-dir" flag.

For example, the below command would list all the different enumerations performed in terms of the given domains and are stored in the "amass4owasp" graph database:

```bash
amass db -dir amass4owasp -list
```

Next, with a command similar to the below retrieve the assets identified during that enumeration -- in this case enumeration 1:

```bash
amass db -dir amass4owasp -d owasp.org -enum 1 -show
```

This is useful when wanting to extract specific information from previous enumerations stored in the Amass database.

### Amass Viz

Amass viz is used to generate visualisations of the data stored in an Amass graph database. The below command would create a 3D visualisation of the data stored in the "amass4owasp" database.

```bash
amass viz -d3 -dir amass4owasp
```

![Amass Visualization](../pics/amass4.png)

### Automation

Here's a script that automates regular scans using Amass and sends notifications via Pushover when new subdomains are discovered:

```bash
APP_TOKEN="$1"
USER_TOKEN="$2"

amass enum -src -active -df ./domains_to_monitor.txt -config ./regular_scan.ini -o ./amass_results.txt -dir ./regular_amass_scan -brute -norecursive
RESULT=$(amass track -df ./domains_to_monitor.txt -config ./regular_scan.ini -last 2 -dir ./regular_amass_scan | grep Found | awk '{print $2}')
FINAL_RESULT=$(while read -r d; do if grep --quiet "$d" ./all_domains.txt; then continue; else echo "$d"; fi; done <<< $RESULT)

if [[ -z "$FINAL_RESULT" ]];
FINAL_RESULT="No new subdomains were found"
else
echo "$FINAL_RESULT" >> ./all_domains.txt
fi
wget https://api.pushover.net/1/messages.json --post-data="token=$APP_TOKEN&user=$USER_TOKEN&message=$FINAL_RESULT&title=$TITLE" -qO- > /dev/null 2>&1 &
```