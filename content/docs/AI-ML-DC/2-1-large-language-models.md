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

### Large Language Model (LLM) - Part 2/2: Transformer Architecture

###  Introduction

Sequence-to-sequence (seq2seq) language translation and Generative Pretrained Transformer (GPT) models are subcategories of Natural Language Processing (NLP) that utilize the Transformer architecture. Seq2seq models are typically using Long Short-Term Memory (LSTM) networks or encoder-decored based Transformers. In contrast, GPT is an autoregressive language model that uses decoder-only Transformer mechanism. The purpose of this chapter is to provide an overview of the decoder-only Transformer architecture.

The Transformer consists of stacks of decoder modules. A word embedding vector, a result of the word tokenization and embbeding, is fed as input to the first decoder module. After processing, the resulting context vector is passed to the next decodeer, and so on. After the final decoder, a softmax layer evaluates the output against the complete vocabulary to predict the next word. As an autoregressive model, the predicted word vector from the softmax layer is converted into a token before being fed back into the subsequent decoder layer. This process involves a token-to-word vector transformation prior to re-entering the decoder.

Each decoder module consists of an attention layer, Add & Normalization layer and a feedforward neural network (FFNN). Rather than feeding the embedded word vector (i.e., token embedding plus positional encoding) directly into the decoder layers, the Transformer first computes the Query (Q), Key (K), and Value (V) vectors from the word vector. These vectors are then used in the self-attention mechanism. Initially, the query vector is multiplied by the key vectors using matrix multiplication. The result is then divided by the square root of the dimension of the key vectors (scaled dot product) to obtain the logits. The logits are processed by a softmax layer to compute probabilities. The SoftMax prediction results are multiplied with the value vectors to produce a context vector.

Before feeding the context vector into the feedforward neural network, it is summed with the original word embedding vector (which includes positional encoding) via a residual connection. Finally, the output is normalized using layer normalization. This normalized output is then passed as input to the FFNN, which computes the output. 

The basic architecture of the FFNN in the decoder is designed so that the input layer has as many neurons as the dimension of the context vector. The hidden layer, in turn, has four times as many neurons as the input layer, while the output layer has the same number of neurons as the input layer. This design guarantees that the output vector of the FFNN has the same dimension as the context vector. Like the attention block, the FFNN block also employs residual connections and normalization.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiA-WSCZiYdJdyVcYiX_Yrbm8yvMeqVk5eA9cyoWsNvDrn4ssrSmvpAYRcRNsgkUk84RMkIF5UMVmIwLeHEn_kDiyOzNxZbjpb2slzC6gfv3fsncJGdIAawcwQYQ8eK6pjCVxg2HI2XjcymG2qlcRbkJRLweXzheGNFOjAm798kNgZzXCL_pCIqhrdSNUA=w640-h390)](https://blogger.googleusercontent.com/img/a/AVvXsEiA-WSCZiYdJdyVcYiX_Yrbm8yvMeqVk5eA9cyoWsNvDrn4ssrSmvpAYRcRNsgkUk84RMkIF5UMVmIwLeHEn_kDiyOzNxZbjpb2slzC6gfv3fsncJGdIAawcwQYQ8eK6pjCVxg2HI2XjcymG2qlcRbkJRLweXzheGNFOjAm798kNgZzXCL_pCIqhrdSNUA)

Figure 7-8: Decoder-Only Transformer Architecture.

  

### Query, Key and Value Vectors

  

As pointed out in the Introduction, the word embedding vector is not used as input to the first decoder. Instead, it is multiplied by pretrained Query, Key, and Value weight matrices. The result of this matrix multiplication, dot product, produces the Query, Key, and Value vectors, which are use as inputs, and are processed through the Transformer. Figure 7–9 show the basic workflow of this process.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEixhuDXW6gNcox0HXLb0pMybU2yfN8EgaOUrPMzOk2QFNzmjtXH5ba1F-_rmOOyoHdy4QtSqiSAVw9f1Md26e1_LXXtXbSB8sZ3XP5vJUMdoIbsxt6BI1RXmp39yXRGBLrDA7r2NBev2fFZ0Xwd5-dwvxIsMJ0ZNsS1J22NRcWeBeKPeENwjFHvrXHYoFo=w640-h372)](https://blogger.googleusercontent.com/img/a/AVvXsEixhuDXW6gNcox0HXLb0pMybU2yfN8EgaOUrPMzOk2QFNzmjtXH5ba1F-_rmOOyoHdy4QtSqiSAVw9f1Md26e1_LXXtXbSB8sZ3XP5vJUMdoIbsxt6BI1RXmp39yXRGBLrDA7r2NBev2fFZ0Xwd5-dwvxIsMJ0ZNsS1J22NRcWeBeKPeENwjFHvrXHYoFo)

Figure 7-9: Query, Key, and Value Vectors.

  

Let’s take a closer look at the process using numbers. After tokenizing the input words and applying positional encoding, we obtain a final 5-dimensional word matrix. To reduce computation cycles, the process reduces the dimension of the Query vector from 5 to 3. Because we want the Query vector to be three-dimensional, we use three 5-dimensional column vectors, each of which is multiplied by the word vector.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEh5J6BY1nCeVA_5MwRB2yWJOcm3yQ0ptuQVa2cXdOuifohAHoE1PlFrnPDgpz6RNL3qhw4QENlQEvS_q1W1zYPxBi_Gm6geAZDBVUTwCCrB6yFxgXD6rTbNZm7TFFOyrdO3kBDSYH_kf4gGZFEluBpmTUy_O2jQ1_e8Lcn4p2JG97AqqoOaM0ze3nEi9e0=w640-h182)](https://blogger.googleusercontent.com/img/a/AVvXsEh5J6BY1nCeVA_5MwRB2yWJOcm3yQ0ptuQVa2cXdOuifohAHoE1PlFrnPDgpz6RNL3qhw4QENlQEvS_q1W1zYPxBi_Gm6geAZDBVUTwCCrB6yFxgXD6rTbNZm7TFFOyrdO3kBDSYH_kf4gGZFEluBpmTUy_O2jQ1_e8Lcn4p2JG97AqqoOaM0ze3nEi9e0)

  

Figure 7-10: Calculating the Query Vector.

  

  

Figure 7–11 depicts the calculation process, where each component of the word vector is multiplied by its corresponding component in the Query weight matrix. The weighted sum of these results forms a three-dimensional Query vector. The Key and Value vectors are calculated using the same method.

  

The same Query, Key, and Value (Q, K, V) weight matrices are used across all words (tokens) within a single self-attention layer in a Transformer model. This ensures that each token is processed in the same way, maintaining consistency in the attention computations. However, each decoder layer in the Transformer has its own dedicated Q, K, and V weight matrices, meaning that every layer learns different transformations of the input tokens, allowing deeper layers to capture more abstract representations.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgrzNs947qFftCKUKKSR-z1GU9UKJo7OHTIzxSA5mBbJk9a2HfbWqzj25Qxg8U8LTeVkYfeONT8nCOxJj7JZdUWd2rFb3UM5nRfRKIXq-4Z6PzWTJl9Exu8ZJVnUVFYjt05YqHAickxbYMo-2dqI9xWajz-Q0M_i3qk4ssfU2UhPY_IioArj-DHfkajxls=w640-h322)](https://blogger.googleusercontent.com/img/a/AVvXsEgrzNs947qFftCKUKKSR-z1GU9UKJo7OHTIzxSA5mBbJk9a2HfbWqzj25Qxg8U8LTeVkYfeONT8nCOxJj7JZdUWd2rFb3UM5nRfRKIXq-4Z6PzWTJl9Exu8ZJVnUVFYjt05YqHAickxbYMo-2dqI9xWajz-Q0M_i3qk4ssfU2UhPY_IioArj-DHfkajxls)

  

Figure 7-11: Calculating the Query Vector.

  

### Attention Layer

  

Figure 7-12 depicts what happens in the first three components of the Attention layer after calculating the Query, Key, and Value vectors. In this example, we focus on the word “clear”, and try to predict the next word. Its Query vector is multiplied by its own Key vector as well as by all the Key vectors generated for the preceding words. Each multiplication produces its own score. Note that the score values shown in the figure are theoretical and are not derived from the actual Qv × Kv matrix multiplication; however, the remaining values are based on these calculations. Additionally, in our example, we use one-dimensional values (instead of actual vectors) to keep the figures and calculations simple. In reality, these are n-dimensional vectors.

  

After the Qv × Kv matrix multiplication, the resulting scores are divided by the square root of the vector depth, yielding logits, i.e., the input values for the softmax function. The softmax function then computes the exponential of each logit (using Euler’s number, approximately 2.71828) and divides each result by the sum of all exponentials. For example, the value 3.16, corresponding to the first word, is divided by 482.22, resulting in a probability of 0.007. Note that the sum of the probabilities is 1.0. Softmax ensures that the attention scores sum to 1, making them interpretable as probabilities and helping the model decide which input tokens to focus on when generating an output. In our example, the token for the word “clear” has the highest probability at this stage. The word “decided” has the second highest probability score (0.210), which indicates that the semantics of “clear”, which has the highest probability score (0.665), can be interpreted as an verb answering the question: “What she decided to do?

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjFW0jelz2WibAXTs-FKUvQ04NM8TLYbPULYwrkQiJk9cTaSIL3PVQuf6Lqkdk2MjSczxYJobMKD9D7wBWmugcNLzcVLdZJew9SiJkMf_aSnOHB-t_EgX4zA9KXU9u8VKXXbhzsDrHjeB70ag3VPvO83Q7WQY347KXHPf967ICJ0anTtxG49-RtgWcz9gQ=w640-h336)](https://blogger.googleusercontent.com/img/a/AVvXsEjFW0jelz2WibAXTs-FKUvQ04NM8TLYbPULYwrkQiJk9cTaSIL3PVQuf6Lqkdk2MjSczxYJobMKD9D7wBWmugcNLzcVLdZJew9SiJkMf_aSnOHB-t_EgX4zA9KXU9u8VKXXbhzsDrHjeB70ag3VPvO83Q7WQY347KXHPf967ICJ0anTtxG49-RtgWcz9gQ)

  

Figure 7-12: Attention Layer, the First Three Steps.

  

Next, the SoftMax probabilities are multiplied by each token's Value vector (matrix multiplication). The resulting vectors are then summed, producing the Context vector for the token associated with the word “clear.” Note that the components of the Value vectors are example values and are not derived from actual computations.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgbjJrw1gfAYzaR8W0Qq4XnmVpuHS32KMBOPBABeX9pxwEpBld7xelB8RGwOaLn1w2iITFxPPwf0Zge6rbJaBB9QfGOSEpY5ynk-YsfE7Mv996GP0mubwtMN8C6ST9eOsWEhbgfou9VR_rVx1cpeHINCc3IT7lG6qpU045YQPek7lA3GtgHaKKD849Amjw=w640-h350)](https://blogger.googleusercontent.com/img/a/AVvXsEgbjJrw1gfAYzaR8W0Qq4XnmVpuHS32KMBOPBABeX9pxwEpBld7xelB8RGwOaLn1w2iITFxPPwf0Zge6rbJaBB9QfGOSEpY5ynk-YsfE7Mv996GP0mubwtMN8C6ST9eOsWEhbgfou9VR_rVx1cpeHINCc3IT7lG6qpU045YQPek7lA3GtgHaKKD849Amjw)

Figure 7-13: Attention Layer, the Fourth Step.

  

### Add & Normalization

  

As the final step, the Word vector, which includes positional encoding, is added to the context vector via a Residual Connection. The result is then passed through a normalization process, where the vector’s components are summed and divided by the vector’s dimension, yielding the mean (μ). This mean value is then used for standard deviation calculation: the mean is subtracted from each of the three vector components, and the results are squared. These squared values are then summed, divided by three (the vector’s dimension), and the square root of this result gives the final output vector \[1.40, -0.55, -0.87\] of the Add & Normalize layer. 

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhk5DccsIEhZB9pcysWsxyRht72JsU7tjY8Cmd0RP1QFu-9l77sy6nI5RlrY5A6gZWhMbprdlmCa9QwqVXQRo8aVXtowCzzrUBtvNDZGEyb0LOOGWj8Jom5IOb9GNjSmfLGLU3phcoEVq2A61zRNVoGu9fO7TPjX8qGFIF_de-ic9SNlAsrFUuVIjxKCEo=w640-h330)](https://blogger.googleusercontent.com/img/a/AVvXsEhk5DccsIEhZB9pcysWsxyRht72JsU7tjY8Cmd0RP1QFu-9l77sy6nI5RlrY5A6gZWhMbprdlmCa9QwqVXQRo8aVXtowCzzrUBtvNDZGEyb0LOOGWj8Jom5IOb9GNjSmfLGLU3phcoEVq2A61zRNVoGu9fO7TPjX8qGFIF_de-ic9SNlAsrFUuVIjxKCEo)

Figure 7-14: Add & Normalize Layer – Residual Connection and Layer Normalization.

  

### Feed Forward Neural Network

Within the decoder module, the feedforward neural network uses the output vector from the Add & Normalize layer as its input. In our example, the FFNN have one neuron in input layer for each component of the vector. This layer simply passes the input values to the hidden layer, where each neuron first calculates a weighted sum and then applies the ReLU activation function. In our example, the hidden layer contains nine neurons (three times the number of input neurons). The output from the hidden layer is then fed to the output layer, where the neurons again compute a weighted sum and apply the ReLU activation function. Note that in transformer-based decoders, the FFNN is applied to each token individually. This means that each token-related output from the attention layer is processed separately by the same FFNN model with shared weights, ensuring a consistent transformation of each token's representation regardless of its position in the sequence.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEi8U4FGV3lVYan5FMglGDCe5YOo21xwMlF0C6sDFIRZ9YDyteq1SOuQiV900o9HfrSObEZr9-37iXqTA3AIT0Yh-9lGz3hxCqdTeB2iHEizitSBG7VFB0PG9UI0i7bUKc7OkdWZIrY4Ka9qe-XLJt3PN6dcpDy2q00AnKOrSdql2WcJ0OvC3UwVMguNeuQ=w640-h340)](https://blogger.googleusercontent.com/img/a/AVvXsEi8U4FGV3lVYan5FMglGDCe5YOo21xwMlF0C6sDFIRZ9YDyteq1SOuQiV900o9HfrSObEZr9-37iXqTA3AIT0Yh-9lGz3hxCqdTeB2iHEizitSBG7VFB0PG9UI0i7bUKc7OkdWZIrY4Ka9qe-XLJt3PN6dcpDy2q00AnKOrSdql2WcJ0OvC3UwVMguNeuQ)

Figure 7-15: Fully Connected Feed Forward Neural Network (FFNN).

  

  

The final decoder output is computed in the Add & Normalize layer, similarly as Add & Normalize after the attention layer. This produces the decoder output, which is used as the input for the next decoder module. 

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEixrA8llC3II66RWYmmSaJFO-Zllzz4U5L65FFIxY5wnAGWBX4ET8ctKL6lzaVGB_8xCXq3KrykCRp_IChXuYG1RIXEgZmMZc0Z9AzWEj9vdd-As08bhU_2WCm44UFz1rh0EQTkJbNiDsT_uCBEZwwHew36oB4ASP6GAe4yoxXxPm3DGfwLXa5VmV7PQ4s=w640-h350)](https://blogger.googleusercontent.com/img/a/AVvXsEixrA8llC3II66RWYmmSaJFO-Zllzz4U5L65FFIxY5wnAGWBX4ET8ctKL6lzaVGB_8xCXq3KrykCRp_IChXuYG1RIXEgZmMZc0Z9AzWEj9vdd-As08bhU_2WCm44UFz1rh0EQTkJbNiDsT_uCBEZwwHew36oB4ASP6GAe4yoxXxPm3DGfwLXa5VmV7PQ4s)

  

Figure 7-16: Add & Normalize Layer – Residual Connection and Layer Normalization.

  

### Next Word Probability Computation – SoftMax Function

  

The output of the last decoder module does not directly represent the next word. Instead, it must be transformed into a probability distribution over the entire vocabulary. First, the decoder output is passed through a weight matrix that maps it to a new vector, where each element corresponds to a word in the vocabulary. For example, in Figure 7-17 the vocabulary consists of 12 words. These words are tokenized and linked to their corresponding word embeddings vector. That said, the word embedding matrix serves as a weight matrix.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiBOZj5xXo4pSpU6l-yehfSvBHYH_eYdaX_7AxQmpiwCyXu_9XDfXJLEpEPaPZ1aA8_dssjzMGsK8Ko9uL3hu4_0ldmojVOXkXMrGcp4vOQZ9ziYwHwUngJziUBh_GhoNwnjg7OY0ONqA1Okf-JiLGPwcGSyQWsU3J7ZTt99PBTQfwrsUGL_Qnu0jWHdK4=w640-h208)](https://blogger.googleusercontent.com/img/a/AVvXsEiBOZj5xXo4pSpU6l-yehfSvBHYH_eYdaX_7AxQmpiwCyXu_9XDfXJLEpEPaPZ1aA8_dssjzMGsK8Ko9uL3hu4_0ldmojVOXkXMrGcp4vOQZ9ziYwHwUngJziUBh_GhoNwnjg7OY0ONqA1Okf-JiLGPwcGSyQWsU3J7ZTt99PBTQfwrsUGL_Qnu0jWHdK4)

Figure 7-17: Hidden State Vector and Word Embedding Matrix.

  

Figure 7-18 illustrates how the decoder output vector (i.e., the hidden state h) is multiplied by all word embedding vectors to produce a new vector of logits. 

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjiJpc8hZy_z_DnazhTGtFUak3H0MM-Dw0DiA6t0gUfL8cghXByrkpR2TZFNAvtlKWeBnN3sTi4ZTYzjBTzOSapdxqjgHetuZL4j0uiLct4qycfEJoBKH3GKFsA0eD2c5mrvcFe3IlQZ7bFqVRbp5a_Rqa4mauj65TudaJtOcX0JRY7-2aOwQIvpAGJ9Xo=w640-h374)](https://blogger.googleusercontent.com/img/a/AVvXsEjiJpc8hZy_z_DnazhTGtFUak3H0MM-Dw0DiA6t0gUfL8cghXByrkpR2TZFNAvtlKWeBnN3sTi4ZTYzjBTzOSapdxqjgHetuZL4j0uiLct4qycfEJoBKH3GKFsA0eD2c5mrvcFe3IlQZ7bFqVRbp5a_Rqa4mauj65TudaJtOcX0JRY7-2aOwQIvpAGJ9Xo)

Figure 7-18: Logits Calculation – Dot Product of Hidden State and Complete Vocabulary.

  

Next, the SoftMax function is applied to the logits. This function converts the logits into probabilities by exponentiating each logit and then normalizing by the sum of all exponentiated logits. The result is a probability distribution in which each value represents the likelihood of selecting a particular word as the next token.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiHO4TOehh_kbfRCZXbTvcd_nEQh3q1cZanXj_eQxAP_DuiTsbT4cQg0clluEwyTA_B1B9kZM1WNIlV1THK067BEn8TMPcpOosJkjgUjyJRK-e_qGu63bvoQ2I3SfTH0Fyp5cmYM7A8SGffdhZkwq8ColVb9N9Wpy62jXL-vw2f5SDPKspDiFQThYCmXug=w640-h366)](https://blogger.googleusercontent.com/img/a/AVvXsEiHO4TOehh_kbfRCZXbTvcd_nEQh3q1cZanXj_eQxAP_DuiTsbT4cQg0clluEwyTA_B1B9kZM1WNIlV1THK067BEn8TMPcpOosJkjgUjyJRK-e_qGu63bvoQ2I3SfTH0Fyp5cmYM7A8SGffdhZkwq8ColVb9N9Wpy62jXL-vw2f5SDPKspDiFQThYCmXug)

  

Figure 7-19: Probability Calculation - Adding Logits to SoftMax Function

  

Finally, the word with the highest probability is selected as the next token. This token is then mapped back to its corresponding word using a token-to-word lookup. This initiates the next iteration, where the token is converted into its word embedding vector, and used together with positional encoding to create the actual word embedding for the next iteration.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEg7Ond-7cxO7_33sU4fnvQt1TYMzUgS9vWkh5A8FuPbErvYt_joaNW5h9iGGC5x4NGVgDqZ41E1eG7dXX-dwoJa2OIwKEJTg9s-MTBX2ETi3R-OO_1scgPghQejvRCWleSCqEsGJpi5LnitRPlTNITo1JC_DqaqnKIDxYrnV69bSQvuXg55h8ayLJgDikE=w640-h368)](https://blogger.googleusercontent.com/img/a/AVvXsEg7Ond-7cxO7_33sU4fnvQt1TYMzUgS9vWkh5A8FuPbErvYt_joaNW5h9iGGC5x4NGVgDqZ41E1eG7dXX-dwoJa2OIwKEJTg9s-MTBX2ETi3R-OO_1scgPghQejvRCWleSCqEsGJpi5LnitRPlTNITo1JC_DqaqnKIDxYrnV69bSQvuXg55h8ayLJgDikE)

  

Figure 7-20: Word-to-Token, and Token-to-Word Embedding Process.

  

In theory, our simple example shows that the model can assign the highest probability to the correct word. For instance, by analyzing the position of the word “clear” relative to its preceding words, the model is able to infer the context. When the context implies that an action is directed toward a known target, the article “the” receives the highest probability score and is predicted as the next word.

  

### Conclusion

  

We use pretty simple examples in this chapter. However, GPT-3, for example, is built on a deep Transformer architecture comprising 96 decoder blocks. Each decoder block is divided into three primary sub-layers:

  

**Attention Layer:** This layer implements multi-head self-attention using four key weight matrices, one each for the query, key, and value projections, plus one for the output projection. Together, these matrices account for roughly _600 million trainable parameters per block_.

  

**Add & Normalize Layers:** Each block employs two residual connections paired with layer normalization. The first Add & Normalize operation occurs immediately after the Attention Layer, and the second follows the Feed-Forward Neural Network (FFNN) layer. Although essential for stabilizing training, the parameters in each normalization step are relatively few, typically on the order of tens of thousands.

  

**Feed-Forward Neural Network (FFNN) Layer:** The FFNN consists of two linear transformations with an intermediate expansion (usually about four times the model’s hidden size). This layer contributes approximately _1.2 billion parameters per block_.

  

Aggregating the parameters from all 96 decoder blocks, and including additional parameters from the token embeddings, positional encodings, and other components, the entire GPT-3 model totals around _175 billion trainable parameters_. This is why parallelism is essential: the training process must be distributed across multiple GPUs and executed according to a selected parallelization strategy. The second part of the book discusses about Parallelization.

**References:**
* https://nwktimes.blogspot.com/2025/02/large-language-models-llm-part-12-word.html
* https://nwktimes.blogspot.com/2025/02/large-language-model-llm-part-22.html
