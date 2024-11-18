# Product Configuration

### PowerDNS module **[WHMCS](https://puqcloud.com/link.php?id=77)** 

#####  [Order now](https://puqcloud.com/index.php?rp=/store/whmcs-module-powerdns) | [Download](https://download.puqcloud.com/WHMCS/servers/PUQ_WHMCS-PowerDNS/) | [FAQ](https://faq.puqcloud.com/)

##### Add new product to WHMCS

```
System Settings->Products/Services->Create a New Product
```

In the **Module settings** section, select the **"PUQ PowerDNS"** module

[![image-1725215821870.png](https://doc.puq.info/uploads/images/gallery/2024-09/scaled-1680-/image-1725215821870.png)](https://doc.puq.info/uploads/images/gallery/2024-09/image-1725215821870.png)

- **License key:** A pre-purchased license key for the **"PUQ PowerDNS"** module. For the module to work correctly, the key must be active
- **Max Zones:** The number of zones that will be available for the client to manage.
- **Edit SOA:** Whether to allow the client to manage the SOA record.
- **Zone name filter:** In this field, you can enter regular expressions to filter zone names that the client can add. Each filter should be on a separate line, and each filter is checked in sequence, meaning the zone will not be added if even one filter matches.
- **Nameservers:** In this section, enter the name servers that will be added to the zone (Your DNS cluster).
- **SOA:** In this section, enter all the SOA record parameters that will be used by default.

#### Zone template 

[![image-1731793137550.png](https://doc.puq.info/uploads/images/gallery/2024-11/scaled-1680-/image-1731793137550.png)](https://doc.puq.info/uploads/images/gallery/2024-11/image-1731793137550.png)

Here are the rules for creating DNS records. These records will be automatically generated when a zone is created. Placeholders like `{zone}` will be replaced with the actual zone name. The format for defining records is as follows:

**Format:**  
`name type ttl content`

### Explanation:

1. **name**:
    
    
    - This specifies the name of the subdomain or record.
    - For example, `ftp` will expand to `ftp.<zone.name>`.
    - Use `@` to refer to the main zone (root domain).
2. **type**:
    
    
    - The type of DNS record.
    - Examples include: `A`, `AAAA`, `MX`, `CNAME`, `TXT`, `SRV`, `CAA`, `DNSKEY`, `DS`, `NAPTR`, `TLSA`.
3. **ttl (Time To Live)**:
    
    
    - The duration (in seconds) for which the record is cached by DNS resolvers.
    - Recommended default is `3600` seconds (1 hour).
4. **content**:
    
    
    - The value or data for the record, provided without abbreviations or placeholders.
    - For example, for an `A` record, this would be the IPv4 address.

These rules ensure consistency and accuracy when defining DNS records for your zones.

#####  

### **Example Zone Records Template**

**A Records (IPv4):**

<div id="bkmrk-%40-a-3600-192.168.1.1">```
@           A       3600    192.168.1.1<br></br>@           A       3600    192.168.1.3<br></br>www         A       3600    192.168.1.2<br></br>www2        A       3600    192.168.1.3
```

</div>**AAAA Records (IPv6):**

<div id="bkmrk-%40-aaaa-3600-2001%3A0db">```plaintext
`@           AAAA    3600    2001:0db8:85a3:0000:0000:8a2e:0370:7334<br></br>@           AAAA    3600    2001:0db8:85a3:0000:0000:8a2e:0370:7336<br></br>www         AAAA    3600    2001:0db8:85a3:0000:0000:8a2e:0370:7335<br></br>www2        AAAA    3600    2001:0db8:85a3:0000:0000:8a2e:0370:7335`
```

</div>**CNAME Records (Aliases):**

<div id="bkmrk-ftp-cname-3600-%7Bzone">```plaintext
`ftp         CNAME   3600    {zone}<br></br>ftp2        CNAME   3600    example.com`
```

</div>**MX Records (Mail Exchange):**

<div id="bkmrk-%40-mx-3600-10-mail.%7Bz">```plaintext
`@           MX      3600    10 mail.{zone}<br></br>@           MX      3600    20 backupmail.{zone}`
```

</div>**TXT Records (Text Data):**

<div id="bkmrk-%40-txt-3600-v%3Dspf1-ip">```plaintext
`@           TXT     3600    v=spf1 ip4:192.168.1.1 -all<br></br>@           TXT     3600    SOME TXT TEXT<br></br>_dmarc      TXT     3600    v=DMARC1; p=none; rua=mailto:dmarc@{zone}`
```

</div>**CAA Records (Certification Authority Authorization):**

<div id="bkmrk-%40-caa-3600-0-issue-l">```plaintext
`@           CAA     3600    0 issue      letsencrypt.org<br></br>@           CAA     3600    0 issuewild  comodoca.com<br></br>@           CAA     3600    0 iodef      mailto:admin@{zone}`
```

</div>**NAPTR Record (Naming Authority Pointer):**

<div id="bkmrk-_sip._udp-naptr-3600">```plaintext
`_sip._udp   NAPTR   3600    100 10 S SIP+D2U * sip.{zone}.`
```

</div>**SRV Records (Service Locator):**

<div id="bkmrk-_sip._tcp-srv-3600-1">```
_sip._tcp   SRV     3600    10 5 5060 sipserver.{zone}
```

</div>- - - - - -

### **Key Notes:**

- **`@`**: Represents the main zone (e.g., the root domain).
- **Placeholders like `{zone}`**: Will be replaced by the actual zone name during execution.
- **TTL (Time to Live)**: Use 3600 seconds by default, which is standard for DNS records.
- Adjust records based on your specific zone requirements. These templates cover common DNS record types for a functional zone configuration.

#####  