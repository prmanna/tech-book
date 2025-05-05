---
title: "Large Language Models (LLM)"
bookCollapseSection: true
weight: 21
---

### Large Language Models (LLM) - Part 1/2: Word Embedding

### Introduction

This chapter introduces the basic operations of Transformer-based Large Language Models (LLMs), focusing on fundamental concepts rather than any specific LLM, such as OpenAI’s GPT (Generative Pretrained Transformer).The chapter begins with an introduction to tokenization and word embeddings, which convert input words into a format the model can process. Next, it explains how the transformer component leverages decoder architecture for input processing and prediction. 

This chapter has two main goals. First, it explains how an LLM understands the context of a word. For example, the word “clear” can be used as a verb (Please, clear the table.) or as an adjective (The sky was clear.), depending on the context. Second, it discusses why LLMs require parallelization across hundreds or even thousands of GPUs due to the large model size, massive datasets, and the computational complexity involved.

  

### Tokenizer and Word Embedding Matrix

As a first step, we import a vocabulary into the model. The vocabulary used for training large language models (LLMs) typically consists of a mix of general and domain-specific terms, including basic vocabulary, technical terminology, academic and formal language, idiomatic expressions, cultural references, as well as synonyms and antonyms. Each word and character is stored in a word lookup table and assigned a unique token. This process is called tokenization.

  

Many LLMs use Byte Pair Encoding (BPE), which splits words into subword units. For example, the word "unhappiness" might be broken down into "un," "happi," and "ness." BPE is widely used because it effectively balances vocabulary size and tokenization efficiency, particularly for handling rare words and sub-words. For simplicity, we use complete words in all our examples.

Figure 7-1 illustrates the relationship between words in the vocabulary and their corresponding tokens. Token values start from 2 because token 0 is reserved for padding and token 1 for unknown words.

Each token, representing a word, is mapped to a Word Embedding Vector, which is initially assigned random values. The collection of these vectors forms a Word Embedding Matrix. The dimensionality of each vector determines how much contextual information it can encode.

For example, consider the word “clear.” A two-dimensional vector may distinguish it as either an adjective or a verb but lacks further contextual information. By increasing the number of dimensions, the model can capture more context and better understand the meaning of the word. In the sentence “The sky was clear,” the phrase “The sky was” suggests that "clear" is an adjective. However, if we extend the sentence to “She decided to clear the backyard of junk,” the word "clear" now functions as a verb. More dimensions allow the model to utilize surrounding words more effectively for next-word prediction. For instance, GPT-3 uses 12,288-dimensional vectors. Given a vocabulary size of 50,000 words used by GPT-3, the Word Embedding Matrix has dimensions of 12,288 × 50,000, resulting in 614,400,000 parameters.

The context size, defined as the sequence length of vectors, determines how many preceding words the model considers when predicting the next word. In GPT-3, the context size is 2,048 tokens.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhE3s3vC-QZBZ5Xdb3q6X0JR8uSa25smRVB0pk-r_IGUGBIb6mnS4CumYCwPYKtqsKK-3nE4qCsVCd3m62iraz124xvdhtkaWs4YgVyfsrH8TjI2iiMsd_XIUpBhSnl_TvyWCxkCb916cXjChKDaQ3DWFXwlcF9p2b1X6qNzoXPa6F0fAH7xsp86cGZU_k=w640-h324)](https://blogger.googleusercontent.com/img/a/AVvXsEhE3s3vC-QZBZ5Xdb3q6X0JR8uSa25smRVB0pk-r_IGUGBIb6mnS4CumYCwPYKtqsKK-3nE4qCsVCd3m62iraz124xvdhtkaWs4YgVyfsrH8TjI2iiMsd_XIUpBhSnl_TvyWCxkCb916cXjChKDaQ3DWFXwlcF9p2b1X6qNzoXPa6F0fAH7xsp86cGZU_k)

  

**Figure 7-1:** _Tokenization and Word Embedding Matrix._

### Word Embedding

As a first step, when we feed input words into a Natural Language Processing (NLP) model, we must convert them into a format the model can understand. This is a two-step process:

1. **Tokenization** – Each word is assigned a corresponding token from a lookup table.
2. **Word Embedding** – These token IDs are then mapped to vectors using a word embedding lookup table.

To keep things simple, Figure 7-2 uses two-dimensional vectors in the embedding matrix. Instead of complete sentences, we use words, which can be categorized into four groups: female, male, adult, and child.

The first word, "Wife," appears in the lookup table with the token value 2. The corresponding word vector in the lookup table for token 2 is \[-4.5, -3.0\]. Note that Figure 7-2 represents vectors as column vectors, whereas in the text, I use row vectors—but they contain the same values.

The second word, "Mother," is assigned the token 3, which is associated with the word vector \[-2.5, +3.0\], and so on.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiJNUFCwHAwH0IzyA13HyM6gLoRNvaW0dXpO-SyB7C3dHaRtK0Vu1Qwk6aAKXNTwi1aZ5pmoHQShSVckIXrs-W_aJ5qGBH_MVPmhHbXswCxPqtrvkOkeBCXtqFe_qYM6vAOLyh5gv36ZfGw_Q1ADLNhhStm4ycchpTvIcpRvwpxtETZhiTzpptddMr4gtE=w640-h362)](https://blogger.googleusercontent.com/img/a/AVvXsEiJNUFCwHAwH0IzyA13HyM6gLoRNvaW0dXpO-SyB7C3dHaRtK0Vu1Qwk6aAKXNTwi1aZ5pmoHQShSVckIXrs-W_aJ5qGBH_MVPmhHbXswCxPqtrvkOkeBCXtqFe_qYM6vAOLyh5gv36ZfGw_Q1ADLNhhStm4ycchpTvIcpRvwpxtETZhiTzpptddMr4gtE)

  

**Figure 7-2:** _Word Tokenization and Word Embedding._

  

In Figure 7-3, we have a two-dimensional vector space divided into four quadrants, representing gender (male/female) and age (child/adult). Tokenized words are mapped into this space.

At the start of the first iteration, all words are placed randomly within the two-dimensional space. During training, our goal is to adjust the word vector values so that adults are positioned on the positive side of the Y-axis and children on the negative side. Similarly, males are placed in the negative space of the X-axis, while females are positioned on the positive side.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhmDPYthV-EqTbhPrj3Dumz5g6wzevO-cwmPartMfq-EEdfOzqmJKYAJfqYxmiQnE657ZZlqxPDSuc5nk7W7lLj7LEkn4BQvx9rWEBZTfM3_84rIq4rtAzs-GlMzuf3p3Srw1HIIby9StqD8Pcmz8MebV4TH_iN8xpAOx7Z-NIs3e6ZWbSQPaZwEt788XY=w640-h460)](https://blogger.googleusercontent.com/img/a/AVvXsEhmDPYthV-EqTbhPrj3Dumz5g6wzevO-cwmPartMfq-EEdfOzqmJKYAJfqYxmiQnE657ZZlqxPDSuc5nk7W7lLj7LEkn4BQvx9rWEBZTfM3_84rIq4rtAzs-GlMzuf3p3Srw1HIIby9StqD8Pcmz8MebV4TH_iN8xpAOx7Z-NIs3e6ZWbSQPaZwEt788XY)

**Figure 7-3:** _Words in the 2 Dimensional Vector Space in the Initial State._

  

Figure 7-4 illustrates how words may be positioned after successful training. All words representing a male adult are placed in the upper-left quadrant (adult/male). Similarly, all other words are positioned in the two-dimensional vector space based on their corresponding age and gender.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiP9XM5LfT6OpI-CLQdQYphnj9jeAeBg3jY2JqlWBAOB4RxvR48uhks43cX3_9Noddz1ObKN1IaPLnlHv2OHU68zifASvm8_y_lCqk7jWAMbalyKB9tXF25NSiHMWOSLDlf0r6N6YYOT8nvzht5tYV1aO1MQgmDaDVOynNqcdMl3iDWLmIKG7ARwyMstTg=w640-h460)](https://blogger.googleusercontent.com/img/a/AVvXsEiP9XM5LfT6OpI-CLQdQYphnj9jeAeBg3jY2JqlWBAOB4RxvR48uhks43cX3_9Noddz1ObKN1IaPLnlHv2OHU68zifASvm8_y_lCqk7jWAMbalyKB9tXF25NSiHMWOSLDlf0r6N6YYOT8nvzht5tYV1aO1MQgmDaDVOynNqcdMl3iDWLmIKG7ARwyMstTg)

  

**Figure 7-4:** _Words in the 2 Dimensional Vector Space After Training._

  

In addition to grouping similar words, such as "adult/female," close to each other in an n-dimensional space, there should also be positional similarities between words in different quadrants. For example, if we calculate the Euclidean distance between the words Father and Mother, we might find that their distance is approximately 4.3. The same pattern applies to word pairs like Nephew-Niece, Brother-Sister, Husband-Wife, and Father-in-Law–Mother-in-Law.

  

However, it is important to note that this example is purely theoretical. In practice, Euclidean distances in high-dimensional word embeddings are not fixed but vary depending on the training data and optimization process. The relationships between words are often better captured through cosine similarity rather than absolute Euclidean distances.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEj_ucF4kW1YBUkvZ82DjUXNnuy2FH-Ltvmnb1zhWyUiS86xZ-ncN3h3CEsBQVMwbwkvzuvARbGz8NEAYIVTWIwz_DZ6OP0u24dK0WdMKRJk7OcXxUWtHeRK9BiTc9na6rD4rnQpd09Mx2OX1ZOV4vS0KluRIqao_tWVd0BHf_kFoI5qgijiHpjpaXZEtR8=w640-h436)](https://blogger.googleusercontent.com/img/a/AVvXsEj_ucF4kW1YBUkvZ82DjUXNnuy2FH-Ltvmnb1zhWyUiS86xZ-ncN3h3CEsBQVMwbwkvzuvARbGz8NEAYIVTWIwz_DZ6OP0u24dK0WdMKRJk7OcXxUWtHeRK9BiTc9na6rD4rnQpd09Mx2OX1ZOV4vS0KluRIqao_tWVd0BHf_kFoI5qgijiHpjpaXZEtR8)

**Figure 7-5:** _Euclidean Distance._

  

### Positional Embeddings

  

Since input text often contains repeated words with different meanings depending on their position, an LLM must distinguish between them. To achieve this, the word embedding process in Natural Language Processing (NLP) incorporates a Positional Encoding Vector alongside the Word Embedding Vector, resulting in the final word representation.

  

In Figure 7-6, the sentence "The sky is clear, so she finally decided to clear the backyard" contains the word clear twice. Repeated words share the same token ID instead of receiving unique ones. In this example, the is assigned token ID 2, and clear is assigned 5. These token IDs are then mapped to vectors using a word embedding lookup table. However, without positional encoding, words with different meanings would share the same vector representation.

  

Focusing on clear (token ID 5), it maps to the word embedding vector \[+2.5, +1.0\] from the lookup table. Since token IDs do not capture word position, identical words always receive the same embedding.

Positional encoding is essential for capturing context and semantic meaning. As shown in Figure 7-6, each input word receives a Positional Encoding Vector (PE) in addition to its word embedding. PE can either be learned and adjusted during training or remain fixed. The final Word Embedding Vector is computed by combining both the Word Embedding Vector and Positional Encoding Vector.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhDazqAestp6Nt6DDKIpfyZwIqfqF0Pr-m_Ca9wrL8YWQT5JytF0u4b-QtaqdjC5gpA2hDWhZzq8p7MeZymgUfYlY-zty1JU5jYewAwm1v0MQJimsKjL0ykj5BBiSwocYQJ3-FX05KuS0b6AFg2Z68LoHFDxbGLvo0GLSZ5rcLMWh3vNa9xACf5Hf4TzGk=w640-h360)](https://blogger.googleusercontent.com/img/a/AVvXsEhDazqAestp6Nt6DDKIpfyZwIqfqF0Pr-m_Ca9wrL8YWQT5JytF0u4b-QtaqdjC5gpA2hDWhZzq8p7MeZymgUfYlY-zty1JU5jYewAwm1v0MQJimsKjL0ykj5BBiSwocYQJ3-FX05KuS0b6AFg2Z68LoHFDxbGLvo0GLSZ5rcLMWh3vNa9xACf5Hf4TzGk)

  

**Figure 7-6:** _Tokenization – Positional Embedding Vector._

  

### Calculating the Final Word Embedding

  

Figure 7-2 presents the equations for computing the final word embedding by incorporating positional embeddings. There are three variables:

  

- **Position (pos)** → The word’s position in the sentence. In our example, the first occurrence of clear is the fourth word, so pos = 4.
- **Dimension (d)** → The depth of the vector. We use a 2-dimensional vector, so d = 2.
- **Index (i)** → Specifies the axis of the vector: 0 for the x-axis and 1 for the y-axis.

The positional embedding is computed using the following equations:

- x-axis: sin(pos/100002i/d), where i = 0
- y-axis: cos(pos/100002i/d, where i = 1

  

For clear at position 4, with d = 2, the resulting 2D positional vector is \[-0.8, +1.0\]. This vector is then added to the input word embedding vector \[+2.5, +1.0\], resulting in the final word embedding vector \[+1.7, +2.0\].

  

Figure 7-7 also shows the final word embedding for the second occurrence of clear, but the computation is omitted.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEg6OQIgMQrC-X5XHfGUwfFxCUqydZXPsrNU91o69ia9K4xOHaD2MGyZ3AubPCNK-obbP_sJS1NyioSfjJHDBWpV7AvzX24esyCVt6vLLVz-lEog6LmAvcLnOf0BFCQbgsBVltN9KFqyyKzO_MA1I9tq1AWiqaEWzkU-V1VlEhTPEHwQ55q79aBio8p21kE=w640-h380)](https://blogger.googleusercontent.com/img/a/AVvXsEg6OQIgMQrC-X5XHfGUwfFxCUqydZXPsrNU91o69ia9K4xOHaD2MGyZ3AubPCNK-obbP_sJS1NyioSfjJHDBWpV7AvzX24esyCVt6vLLVz-lEog6LmAvcLnOf0BFCQbgsBVltN9KFqyyKzO_MA1I9tq1AWiqaEWzkU-V1VlEhTPEHwQ55q79aBio8p21kE)


**Figure 7-7:** _Finals Word Embedding for the 4th Word._

**References:**
