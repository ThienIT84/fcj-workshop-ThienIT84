---
title: "Blog 2"
date: 2026-06-09
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# CLICKFORCE ACCELERATES AD ANALYTICS WITH AMAZON BEDROCK AGENTS

In the context of the growing development of Generative AI and AI Agents, I find that one of the most practical problems businesses are facing is: manual ad data analysis is too slow and can't keep up with decision-making speed.

![CLICKFORCE Lumos System Architecture](/images/3-BlogsTranslated/blog2.png)

CLICKFORCE — a leading digital advertising company in Taiwan — used to need 2–6 weeks just to create an industry analysis report, because they had to collect data, process, analyze and synthesize insights through many manual steps.

To solve this problem, they built the Lumos system by combining Amazon Bedrock Agents, Amazon Bedrock Knowledge Bases, Amazon SageMaker AI, Amazon OpenSearch Service, AWS Glue, AWS Lambda, and Amazon ECS.

The solution helps automate the entire analysis process, allowing users to query data in natural language and create intelligent reports in a much shorter time.

---

## SOME HIGHLIGHTS

- **Building an AI Agent for Data Analysis:**
  Amazon Bedrock Agents help the system understand user requests, plan processing, and call appropriate tools to analyze data.

- **Reducing Hallucination with Knowledge Bases:**
  Amazon Bedrock Knowledge Bases help the system retrieve information from real data such as industry reports, campaign materials, and advertising data — instead of letting the LLM "make up" answers.

- **Automated Data Querying with Text-to-SQL:**
  Users ask questions in natural language, and the system automatically converts them into SQL queries to get exactly the data they need. No SQL knowledge required.

- **Optimizing Accuracy with Amazon SageMaker AI:**
  SageMaker AI is used to continuously monitor, evaluate, and improve the Text-to-SQL pipeline, helping query results become more accurate and stable over time.

- **More Efficient Data Processing and Retrieval:**
  AWS Glue handles data processing and standardization, while Amazon OpenSearch Service handles indexing and quick search when needed.

- **Optimizing Cost and Operating Time:**
  After deployment, CLICKFORCE reduced report creation time from 2–6 weeks to less than 1 hour, while also reducing operating costs by 47%.

---

## THE NEW ERA DATA ANALYSIS APPROACH

Instead of relying entirely on the data analyst team or having to write manual queries, users can ask questions directly in natural language.

Amazon Bedrock Agents will understand the request, identify the appropriate data source, call the right tools, and return analysis results.

For example, the marketing team can ask:

"Which campaign performed best this month?"

"Which customer group responded most positively to the new ad?"

The system automatically retrieves data, analyzes results, and helps create insights for faster decision-making.

---

## CONCLUSION

What I take away from this case study: Amazon Bedrock Agents are not only suitable for building chatbots, but can also become an AI Agent that truly supports enterprise data analysis.

When combined with Knowledge Bases, SageMaker AI, OpenSearch, Glue, and Lambda, businesses can shorten analysis time, reduce operating costs, and exploit data more effectively.
