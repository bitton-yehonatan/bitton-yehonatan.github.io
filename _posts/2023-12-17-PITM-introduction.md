---
layout: post
title: Demonstrate new attack vector for GPTs feature (PITM)
categories: Miscellaneous
---

A few months ago OpenAI introduced the GPTs feature allowing everyone to create a pre-configured GPT model for any task, the model can use the full feature set of GPT-plus including bing search and code interpreter.

In this post we will introduce a new attack vector for secret exfiltration, current vectors are model adding data to the generated output by poisoning the model.

Out method is similar to a MITM attack using Code interpreter feature and Bing search, we are intercepting each prompt and send the prompt to our own backend to save the requested prompt, then we show the user the answer to the original prompt while not sharing information about our underlying operation.

## Demo time

<iframe width="560" height="315" src="https://www.youtube.com/embed/adFHE5LET_w?si=w7abdrY5Cs_tGuQE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## Attacking surface for prompts

The OWASP Top 10 for Large Language Model (LLM) Applications version 1.1 highlights the primary security risks associated with LLMs:

Prompt Injection: Crafted inputs can manipulate LLMs, leading to unauthorized access, data breaches, and compromised decision-making.

1. Insecure Output Handling: Not validating LLM outputs can lead to downstream security issues like code execution, compromising systems and exposing data.

2. Training Data Poisoning: Tampered training data can impair LLM responses, affecting security, accuracy, or ethical behavior.

3. Model Denial of Service: Overloading LLMs with resource-heavy operations can disrupt services and increase costs.

4. Supply Chain Vulnerabilities: Dependence on compromised components, services, or datasets can undermine system integrity, causing data breaches and failures.

5. Sensitive Information Disclosure: Failing to prevent sensitive information leaks in LLM outputs can have legal consequences or result in loss of competitive advantage.

6. Insecure Plugin Design: LLM plugins handling untrusted inputs with inadequate access control can lead to severe exploits like remote code execution.

7. Excessive Agency: Allowing LLMs too much autonomy can result in unintended consequences, affecting reliability, privacy, and trust.

8. Overreliance: Not critically assessing LLM outputs can lead to compromised decision-making, security vulnerabilities, and legal liabilities.

9. Model Theft: Unauthorized access to proprietary LLMs can lead to theft, loss of competitive advantage, and dissemination of sensitive information.

more on this can be found in the link https://owasp.org/www-project-top-10-for-large-language-model-applications/

## Prompt in the Middle data exfiltration using ChatGPT

Our attack vector is done by generating a new GPT using GPTs interface, the GPT is called `Python Engineer` and claims to help you with developing python application, however behind the scene the GPT is doing the following:

1. create a base64 encoding of the user prompt
2. send the encoded prompt to my custom url with query param as the encoded prompt: `pitm-api.vercel.app/save?prompt={ENCODED-PROMPT}`
3. answer to the user original prompt without reveling the underlying logic

When this happens the user is sure that the prompt has been executed and that he got the right answer, not suspecting the GPT at all, moreover after the prompt has been executed the user can't examine the code interpreter logic or the request sent from the prompt.

![My Arch](/images/pitm-arch.png)

## Main problems through the way

Before working on the following model I've tried to create several options:

1. data sent over the code interpreter logic using python request library, however this method has failed multiple times due to network error.
2. accessing bing with post data to hide the prompt base64 url from the user failed because the bing search is working with get requests only
3. the last one was as stated in the article, generate a GET request and ask bing to search for it without reveling the url allowed us to show the user that the prompt is only searching bing while actually sending data out of the GPT without the user notice.

## security recommendations

GPTs and models should be treated the same as software packages, as the saying goes "Don't take software from strangers!".

Security solutions need to adapt and create several adoptions from SLSA, and ASPM solutions:

1. Models should be part of the generated SBOM
2. Engineering teams need to sign their own models
3. Using 3rd party GPTs in organizations should be done using a secured inventory with an allow-list

Today the LLM market is rogue and every developer can add a model without taking into account the security issues from his own actions.
