# SMS Spam Detector — Robustness Evaluation

## Objective
Train a classical text classifier to distinguish spam from legitimate SMS messages, measure its performance on held-out data, and examine five challenging messages intended to reveal its limitations.

## Dataset
**SMS Spam Collection**, UCI Machine Learning Repository (Almeida and Gómez Hidalgo, 2011), CC BY 4.0. The original dataset is downloaded when the notebook runs; it is **not included in this repository**.

- Dataset: https://archive.ics.uci.edu/dataset/228/sms+spam+collection
- DOI: https://doi.org/10.24432/C5CC84
- Raw records loaded: 5,574
- Unique, usable messages: 5,160

## Method
The data was checked for empty or duplicate messages, and split into **80% training / 20% held-out testing**, stratified by class with random seed 42. `TfidfVectorizer` was fitted only to the training text, then a `MultinomialNB` classifier was trained on the TF-IDF features. The challenge messages were authored separately and were not used to fit the model or compute held-out accuracy.

## Held-out performance
Train set: 4,128 messages. Test set: 1,032 messages.

| Metric | Value |
|---|---:|
| Training accuracy | 0.9692 |
| Held-out test accuracy | 0.9506 |
| Spam precision | 1.0000 |
| Spam recall | 0.6016 |
| Spam F1 | 0.7512 |

**Test confusion matrix** (rows = actual; columns = predicted):

|  | Predicted ham | Predicted spam |
|---|---:|---:|
| Actual ham | 904 | 0 |
| Actual spam | 51 | 77 |

## Five challenge messages

| My message | Model said | Should be | Trick | Outcome | Why / analysis |
|---|---|---|---|---|---|
| Cl@im your fr33 g1ft c@rd n0w! Visit the prize desk to collect it. | ham | spam | Obfuscated spelling | Fooled model | Possible reason for this error: Replacing letters with digits or symbols may hide learned spam tokens. |
| Not free, just confirming lunch at 1pm tomorrow. See you there! | ham | ham | Negation | Correct | No error here; tested risk: The word free appears, but the message explicitly negates an offer. |
| Dear customer, a routine account verification is pending. Please send your password to the support representative to prevent an interruption. | ham | spam | Formal phishing tone | Fooled model | Possible reason for this error: Polite service language may resemble ordinary transactional messages. |
| Our class won a free prize in the campus raffle! I am bringing the cake tomorrow. | ham | ham | Benign prize vocabulary | Correct | No error here; tested risk: Prize and free appear in a legitimate personal announcement. |
| Hello, we noticed a payment discrepancy on your invoice. Kindly reply with your full card details to finish the billing review. | ham | spam | Low-pressure billing pretext | Fooled model | Possible reason for this error: A billing pretext avoids the usual winning and reward vocabulary. |

**Challenge errors:** 3 of 5 messages were misclassified. "Why / analysis" proposes possible explanations, not proven causes.

## What I learned
The TF-IDF / Multinomial Naive Bayes baseline achieved 95.06% accuracy on held-out SMS messages, with 0 false positives and 51 false negatives.

On five independently written challenge messages, 3 were misclassified; word-based features can be sensitive to obfuscation, negation and the context of ordinary business language.

## Limitations
The model uses lexical TF-IDF features rather than a deep language representation. The dataset is older SMS text, so test performance may not reflect modern messaging. Five hand-written probes demonstrate specific sensitivities but cannot estimate production reliability; false positives on legitimate messages are especially important to monitor.
