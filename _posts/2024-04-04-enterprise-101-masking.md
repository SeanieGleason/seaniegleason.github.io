---
layout: post
title: Masking PII in Java Logs
subtitle: Using logback to mask PII in all your application logs.
share-img: /assets/img/facial-recognition.png
---

**Keeping customers data safe is one of the most important jobs of building software.**

## PII Data Security

![](../assets/img/facial-recognition.png)

**PII, or Personal Identifiable Information,** has a broad spectrum of meaning in the United States.

The U.S. Department of Labor defines them as:

"Any representation of information that permits the identity of an individual to whom the information applies to be reasonably inferred by either direct or indirect means," [(i)](https://www.dol.gov/general/ppii).

They list examples such as **name, address, social security number** or other **identifying number or code, telephone number, or email address**.
They do further clarify that it does include any data **which an agency intends to identify specific individuals in conjunction with other data elements,**
but alot is left to interpretation.

**_Even though this type of data is defined in the USA, if you are an adult in the United States, your PII data is only partially legally protected._**  

While Europe has [GDPR](https://gdpr-info.eu/), we only target specific sectors for PII, specifically [health data](https://www.hhs.gov/hipaa/index.html) and [student data](https://www2.ed.gov/policy/gen/guid/fpco/ferpa/index.html).
**Every other PII data point is not required to be protected by any laws in the United States.**

*With the laxnesss of these laws compared to the rest of the world, it has become imperative, if not our moral duty as engineers, 
to mask PII data before sharing it with other companies.*

## Application Log Masking
Every application running on the web produces logs.  These logs include business functions and processes and
are needed for monitoring performance and troubleshooting.  However useful this data is for developers to debug,
this type of logging often includes PII data of customers. 

**_These application logs are usually collected into 3rd party Log Aggregator application
(i.e. [Splunk](https://en.wikipedia.org/wiki/Splunk[Splunk)), and by not masking PII data in our logs,
we have indirectly included our customers PII data in any data breaches experienced by those 3rd parties._**

To correct this, regex solutions can be applied to most logging frameworks to turn those fields into gibberish before being logged.

## Java Enterprise Solution : Logback + Logstash Encoder + Regex

Java has a robust logging API known as Logback.
Along with one of its extension projects, Logstash Logback Encoder, we can perform some powerful masking operations to our logs.

Let us take a look at an example:
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.JsonEncoder"/>
    </appender>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder name="STDOUT" class="net.logstash.logback.encoder.LogstashEncoder">
            <jsonGeneratorDecorator class="net.logstash.logback.mask.MaskingJsonGeneratorDecorator">
                <valueMask>
                    <value>(?i)(\s*)(|"|')(name|phoneNumber|race|gender|birthdate|geodata)(|"|')(\s*)(:|=)(\s*)('|"|\s*)(\s*)(^$|.*?)(\s*)('|,|"|\))</value>
                    <mask>$1$2$3$4$5$6$7$8$9*******$11$12</mask>
                </valueMask>
                <valueMask>
                    <value>(?i)([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*@([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*\.([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*</value>
                    <mask>$1******@$2******.$3**</mask>
                </valueMask>
                <valueMask>
                    <value>(\d+\.\d+\.\d+\.\d+)</value>
                    <mask>***.***.*.*</mask>
                </valueMask>
            </jsonGeneratorDecorator>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

The first and last pieces tell Logback to use Json Output, and set the log level to <code>DEBUG</code>:
```xml

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.JsonEncoder"/>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
```

The `MaskingJsonGeneratorDecorator` section is the most interesting part.  
Using the Logback Encoder library, we mask a set of PII fields based on predefinened regexs.

The first two are the easiest to understand:
```xml
<configuration>
 ...
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">

        <encoder name="STDOUT" class="net.logstash.logback.encoder.LogstashEncoder">
            <jsonGeneratorDecorator class="net.logstash.logback.mask.MaskingJsonGeneratorDecorator">
                <valueMask>
                    <value>(\d+\.\d+\.\d+\.\d+)</value>
                    <mask>***.***.*.*</mask>
                </valueMask>
                <valueMask>
                    <value>(?i)([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*@([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*\.([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*</value>
                    <mask>$1******@$2******.$3**</mask>
                </valueMask>
                <valueMask>
                    <value>(?i)(\s*)(|"|')(name|phoneNumber|race|gender|birthdate|geodata)(|"|')(\s*)(:|=)(\s*)('|"|\s*)(\s*)(^$|.*?)(\s*)('|,|"|\))</value>
                    <mask>$1$2$3$4$5$6$7$8$9*******$11$12</mask>
                </valueMask>
            </jsonGeneratorDecorator>
        </encoder>
    </appender>
...
</configuration>
```

**IP Addresses** - `(\d+\.\d+\.\d+\.\d+)` targets IP Addresses, substituting digits (\d) for '*' in standard ip format.
You can test it [here](https://regex101.com/r/45heZ9/1).

**Email Addresses** - `(?i)([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*@([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*\.([a-zA-Z0-9. _%+-])[a-zA-Z0-9. _%+-]*` targets email addresses, substituting ever letter except the first of each subsection of the email address.
You can test it [here](https://regex101.com/r/RqzU10/1).

**Enterprise PII** - The last configuration is the one that requires the most coordination across an enterprise.
We are specifically targeting PII fields on business POJO's as they are being printed, for example:

<code><b>{"username" : "myName"} -> {"username" : "******"}*</b></code>

But the regex won't target this:
<code><b>{"someField" : "username"} -> {"someField" : "username"}</b></code>.

Again, you can test it [here](https://regex101.com/r/WggYpa/1).

## Masking Logs - Responsibility
Java has become a powerful language for enterprise web use, but with that power comes the responsibility to mask your customers' PII data. 
If you're a software engineer residing in the United States and don't observe your enterprise employing similar masking techniques, bring it up to your security team!

I will continue this series on masking with a follow-up that will cover how to mask fields in HTTP REST Responses. 
Until next time!