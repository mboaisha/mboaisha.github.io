---
layout: post
title: ChatGPT applications for blue teams
---
Even though the early demo of ChatGPT was released in December of last year, the conversational chatbot is still seeing experimentation by users. In this blog post, I explore some use cases I found.

## What is ChatGPT?
ChatGPT is a conversational AI language model developed by OpenAI. It is trained on a massive amount of text data and can generate human-like responses to text-based prompts. It can be used for various natural language processing tasks such as text generation, question answering, and language translation.

The first GPT model was introduced in 2018 and was trained on a large corpus of text data, using a deep learning technique called Transformer. The success of the first GPT model led to the development of larger and more advanced models, including GPT-2 and GPT-3, which have set new benchmarks for language understanding and generation tasks.

## Limitations
Computers are generally *dumb* but sure are fast. I found that the best results that I got from ChatGPT are when I am extremely specific with my questions. Just like how search engines can give you anything useful if you search for something that's generalized or common, the chatbot will act the same way.

Even though ChatGPT is extremely convincing and human-like, it's answers should be verified. It is possible for ChatGPT to give a misinformed answer.

## Use Cases
### Generating templates for reports
As a cyber threat intelligence analyst, I have to write reports and intelligence products for a myriad of audiences. ChatGPT helped me get ideas on how certain reports should look like. 

**Question:**  Give me a sample of a threat intelligence report that a CISO would want to read.

**Answer:**
```
## Date: 2023-02-03

### Overview

The purpose of this report is to provide a summary of the latest threats and vulnerabilities that could impact the organization. This report is designed to give a clear and concise overview of the threat landscape to help the Chief Information Security Officer (CISO) and their team make informed decisions about the security of their systems.

### Threats

1.  **Ransomware attacks**: Ransomware attacks have continued to increase in frequency and sophistication in recent months. These attacks typically involve attackers encrypting an organization's data and demanding payment in exchange for the decryption key.
    
2.  **Phishing attacks**: Phishing attacks remain a major threat to organizations, with attackers using social engineering techniques to trick users into revealing sensitive information or downloading malware.
    
3.  **Cloud security**: The increasing use of cloud computing has led to new security challenges, as organizations must now secure data that is stored outside their network. This includes threats such as unauthorized access to data, data breaches, and misconfigured cloud environments.
    

### Vulnerabilities

1.  **Unpatched software**: Unpatched software remains one of the most common vulnerabilities, with many organizations failing to regularly update their systems to address known security weaknesses.
    
2.  **Outdated software**: Outdated software also presents a significant security risk, as it is often no longer supported and may contain unpatched vulnerabilities that can be exploited by attackers.
    
3.  **Misconfigured systems**: Misconfigured systems can also pose a significant security risk, as they may contain open ports, weak passwords, and other security weaknesses that can be easily exploited by attackers.
    

### Recommendations

1.  Implement regular patch management to ensure all software is up-to-date and secure.
    
2.  Conduct regular security assessments to identify and address vulnerabilities in the network.
    
3.  Educate employees on the importance of security awareness and best practices to reduce the risk of phishing attacks and other types of social engineering.
    
4.  Implement a cloud security strategy to ensure that data is properly secured in the cloud.
    
5.  Work with security vendors to identify and address new threats as they emerge.
    

### Conclusion

This report provides an overview of the current threat landscape and the vulnerabilities that organizations must be aware of. By taking a proactive approach to security, organizations can reduce the risk of attacks and protect their sensitive information.
```

Here we go... that's a good baseline to go from.

### Malware analysis "assistant"
In the course of my work, I am often tasked with analyzing and reverse engineering malware, somtimes it's nice to have an extra set of eyes on a malicious piece of code.

In this example, I was analyzing a phish kit that had an obfuscated Javascript file. After peeling one layer of obfuscation, I threw it in ChatGPT and asked it to explain to me the AJAX request functions that were present in the code.

![mal-analysis-chatgpt](/assets/images/mal-analysis-chatgpt.png)
### Automation creation
![xkcd-1319](/assets/images/xkcd-1319.png)
[Courtesy of xkcd](https://xkcd.com/1319)

I am not exactly a Python or shell scripting wizard (yet), as such, it takes me a long time to come up with a proper script that does what I want, especially if I want to automate a mundane and repetitive task.

**Question:** Can you write a Python script so that when I give it IoCs, it will write them out as a Splunk query?

**Answer: **
![python-task1](/assets/images/python-task1.png)

### Search engine, but better
At this point, I am convinced that ChatGPT is going to be the next logical evolutionry step for search engines. I can ask it questions and it will give me answers. Often times as a CTI analyst, I am asked to translate technical concepts to strategic audiences where needed, as such, I can come up with a question like this:

**Question:** Explain like I am five years-old how DHCP works 

**Answer:**
```
Okay, imagine you have a big family and you all live in a big house with lots of rooms. In this house, every room has its own TV, but you only have one cable box that provides all the channels.

Now, when you want to watch TV in your room, you have to go to the cable box and change the channel for your TV. But that's not fun, right? So, your parents came up with a solution. They made a helper, let's call him Mr. DHCP, who will do this job for you.

Mr. DHCP will take care of changing the channels for all the TVs in the house, so you can just relax and watch your show. Whenever you want to watch TV, you just have to tell Mr. DHCP what you want to watch and he'll make sure that your TV gets the right channel.

This is similar to how your computer gets information about how to connect to the internet from your router. The router acts like Mr. DHCP and gives your computer all the information it needs to connect to the internet, like the IP address it should use, the name of the internet service provider, etc.

In this way, your computer can easily connect to the internet without you having to remember all the different settings every time. That's what DHCP does!
```

This answer seems to get the point across and simplifies DHCP in terms anyone could understand.

## Looking ahead

Even though ChatGPT is mere chatbot, it can still do powerful things in context of cybersecurity. I suspect that organizations with the capability and expertise to utilize AI technologies would benefit greatly with incorporating the language model into their products.
