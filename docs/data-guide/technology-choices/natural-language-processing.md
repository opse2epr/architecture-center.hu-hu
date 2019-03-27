---
title: A természetes nyelvi feldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 918ac19aeba36ce695c30896113a4c00e2f7c488
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299222"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="e36ea-102">Az Azure-ban technológia feldolgozása természetes nyelv kiválasztása</span><span class="sxs-lookup"><span data-stu-id="e36ea-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="e36ea-103">Szabad formátumú szöveges feldolgozási hajtja végre az szöveg-, keresési, támogatása céljából általában bekezdéseit tartalmazó dokumentumok, de is szolgál, mint a hangulatelemzés, témafelismerés, természetes nyelvi feldolgozást (NLP) feladatait más nyelv észlelése, a kulcsszókeresést és kategorizálási dokumentum.</span><span class="sxs-lookup"><span data-stu-id="e36ea-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="e36ea-104">Ez a cikk a technológiai lehetőségekhez támogatásához a NLP feladatokat-kiszolgálóként működő összpontosít.</span><span class="sxs-lookup"><span data-stu-id="e36ea-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="e36ea-105">Mik azok a beállítások egy NLP szolgáltatás kiválasztásakor?</span><span class="sxs-lookup"><span data-stu-id="e36ea-105">What are your options when choosing an NLP service?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="e36ea-106">Az Azure-ban a következő szolgáltatásokat nyújtanak a természetes nyelvi feldolgozási (NLP) képességek:</span><span class="sxs-lookup"><span data-stu-id="e36ea-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="e36ea-107">Az Azure HDInsight Spark-és Spark MLlib</span><span class="sxs-lookup"><span data-stu-id="e36ea-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="e36ea-108">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="e36ea-108">Azure Databricks</span></span>](/azure/azure-databricks/what-is-azure-databricks)
- [<span data-ttu-id="e36ea-109">A Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="e36ea-109">Microsoft Cognitive Services</span></span>](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a><span data-ttu-id="e36ea-110">Fontosabb kritériumok</span><span class="sxs-lookup"><span data-stu-id="e36ea-110">Key selection criteria</span></span>

<span data-ttu-id="e36ea-111">Így szűkítheti, első lépésként a kérdések megválaszolása:</span><span class="sxs-lookup"><span data-stu-id="e36ea-111">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="e36ea-112">Biztosan előre összeállított modellek használata?</span><span class="sxs-lookup"><span data-stu-id="e36ea-112">Do you want to use prebuilt models?</span></span> <span data-ttu-id="e36ea-113">Ha igen, érdemes a Microsoft Cognitive Services által kínált az API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="e36ea-113">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="e36ea-114">Szükség van egy nagy forrásgyűjteményébe álló szöveges adatok elleni egyéni modelleket taníthat be?</span><span class="sxs-lookup"><span data-stu-id="e36ea-114">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="e36ea-115">Ha igen, fontolja meg az Azure HDInsight Spark MLlib és a Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="e36ea-115">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="e36ea-116">A jogkivonatok, származtató, morfológiai és kifejezés gyakoriság/inverz dokumentum gyakorisága (TF/IDF) alacsony szintű NLP képességek szükségesek?</span><span class="sxs-lookup"><span data-stu-id="e36ea-116">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="e36ea-117">Ha igen, fontolja meg az Azure HDInsight Spark MLlib és a Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="e36ea-117">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="e36ea-118">Egyszerű, magas szintű NLP szolgáltatásokat, például az entitás- és leképezés azonosítása, témafelismerés, helyesírás-ellenőrzés vagy hangulatelemzés kell?</span><span class="sxs-lookup"><span data-stu-id="e36ea-118">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="e36ea-119">Ha igen, érdemes a Microsoft Cognitive Services által kínált az API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="e36ea-119">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="e36ea-120">Képességmátrix</span><span class="sxs-lookup"><span data-stu-id="e36ea-120">Capability matrix</span></span>

<span data-ttu-id="e36ea-121">A következő táblázat összefoglalja a fő különbségeket, a képességek.</span><span class="sxs-lookup"><span data-stu-id="e36ea-121">The following tables summarize the key differences in capabilities.</span></span>

### <a name="general-capabilities"></a><span data-ttu-id="e36ea-122">Általános képességek</span><span class="sxs-lookup"><span data-stu-id="e36ea-122">General capabilities</span></span>

| | <span data-ttu-id="e36ea-123">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="e36ea-123">Azure HDInsight</span></span> | <span data-ttu-id="e36ea-124">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="e36ea-124">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e36ea-125">Modellek imagenet kínál szolgáltatásként</span><span class="sxs-lookup"><span data-stu-id="e36ea-125">Provides pretrained models as a service</span></span> | <span data-ttu-id="e36ea-126">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-126">No</span></span> | <span data-ttu-id="e36ea-127">Igen</span><span class="sxs-lookup"><span data-stu-id="e36ea-127">Yes</span></span> |
| <span data-ttu-id="e36ea-128">REST API</span><span class="sxs-lookup"><span data-stu-id="e36ea-128">REST API</span></span> | <span data-ttu-id="e36ea-129">Igen</span><span class="sxs-lookup"><span data-stu-id="e36ea-129">Yes</span></span> | <span data-ttu-id="e36ea-130">Igen</span><span class="sxs-lookup"><span data-stu-id="e36ea-130">Yes</span></span> |
| <span data-ttu-id="e36ea-131">Programozhatóság</span><span class="sxs-lookup"><span data-stu-id="e36ea-131">Programmability</span></span> | <span data-ttu-id="e36ea-132">Python, Scala, Java</span><span class="sxs-lookup"><span data-stu-id="e36ea-132">Python, Scala, Java</span></span> | <span data-ttu-id="e36ea-133">C#, Java, Node.js, Python, PHP, Ruby</span><span class="sxs-lookup"><span data-stu-id="e36ea-133">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="e36ea-134">Big data jellegű adatkészletek és a nagy dokumentumok feldolgozásának támogatásához</span><span class="sxs-lookup"><span data-stu-id="e36ea-134">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="e36ea-135">Igen</span><span class="sxs-lookup"><span data-stu-id="e36ea-135">Yes</span></span> | <span data-ttu-id="e36ea-136">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-136">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="e36ea-137">Az alsó szintű természetes nyelvi feldolgozási képességek</span><span class="sxs-lookup"><span data-stu-id="e36ea-137">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="e36ea-138">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="e36ea-138">Azure HDInsight</span></span> | <span data-ttu-id="e36ea-139">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="e36ea-139">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- |
| <span data-ttu-id="e36ea-140">Jogkivonatokat létrehozó</span><span class="sxs-lookup"><span data-stu-id="e36ea-140">Tokenizer</span></span> | <span data-ttu-id="e36ea-141">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-141">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-142">Igen (nyelvi elemzési API-t)</span><span class="sxs-lookup"><span data-stu-id="e36ea-142">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="e36ea-143">Szótőkereső</span><span class="sxs-lookup"><span data-stu-id="e36ea-143">Stemmer</span></span> | <span data-ttu-id="e36ea-144">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-144">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-145">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-145">No</span></span> |
| <span data-ttu-id="e36ea-146">Lemmatizer</span><span class="sxs-lookup"><span data-stu-id="e36ea-146">Lemmatizer</span></span> | <span data-ttu-id="e36ea-147">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-147">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-148">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-148">No</span></span> |
| <span data-ttu-id="e36ea-149">Szövegrészeket része</span><span class="sxs-lookup"><span data-stu-id="e36ea-149">Part of speech tagging</span></span> | <span data-ttu-id="e36ea-150">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-150">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-151">Igen (nyelvi elemzési API-t)</span><span class="sxs-lookup"><span data-stu-id="e36ea-151">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="e36ea-152">Kifejezés gyakoriság/inverz-dokumentum gyakorisága (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="e36ea-152">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="e36ea-153">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="e36ea-153">Yes (Spark MLlib)</span></span> | <span data-ttu-id="e36ea-154">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-154">No</span></span> |
| <span data-ttu-id="e36ea-155">Karakterlánc-hasonlóság&mdash;távolság számítási szerkesztése</span><span class="sxs-lookup"><span data-stu-id="e36ea-155">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="e36ea-156">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="e36ea-156">Yes (Spark MLlib)</span></span> | <span data-ttu-id="e36ea-157">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-157">No</span></span> |
| <span data-ttu-id="e36ea-158">N-Gram típusú számítási</span><span class="sxs-lookup"><span data-stu-id="e36ea-158">N-gram calculation</span></span> | <span data-ttu-id="e36ea-159">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="e36ea-159">Yes (Spark MLlib)</span></span> | <span data-ttu-id="e36ea-160">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-160">No</span></span> |
| <span data-ttu-id="e36ea-161">Állítsa le a word eltávolítása</span><span class="sxs-lookup"><span data-stu-id="e36ea-161">Stop word removal</span></span> | <span data-ttu-id="e36ea-162">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="e36ea-162">Yes (Spark MLlib)</span></span> | <span data-ttu-id="e36ea-163">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-163">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="e36ea-164">Magas szintű természetes nyelvi feldolgozási képességek</span><span class="sxs-lookup"><span data-stu-id="e36ea-164">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="e36ea-165">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="e36ea-165">Azure HDInsight</span></span> | <span data-ttu-id="e36ea-166">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="e36ea-166">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e36ea-167">Entitás/szándékot azonosítása és kinyerése</span><span class="sxs-lookup"><span data-stu-id="e36ea-167">Entity/intent identification and extraction</span></span> | <span data-ttu-id="e36ea-168">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-168">No</span></span> | <span data-ttu-id="e36ea-169">Igen (Language Understanding Intelligent Service (LUIS) API)</span><span class="sxs-lookup"><span data-stu-id="e36ea-169">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |
| <span data-ttu-id="e36ea-170">Téma azonosítása</span><span class="sxs-lookup"><span data-stu-id="e36ea-170">Topic detection</span></span> | <span data-ttu-id="e36ea-171">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-171">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-172">Igen (szövegelemzési API-val)</span><span class="sxs-lookup"><span data-stu-id="e36ea-172">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="e36ea-173">Helyesírás-ellenőrzés</span><span class="sxs-lookup"><span data-stu-id="e36ea-173">Spell checking</span></span> | <span data-ttu-id="e36ea-174">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-174">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-175">Igen (a Bing Spell Check API bemutatása)</span><span class="sxs-lookup"><span data-stu-id="e36ea-175">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="e36ea-176">Hangulatelemzés</span><span class="sxs-lookup"><span data-stu-id="e36ea-176">Sentiment analysis</span></span> | <span data-ttu-id="e36ea-177">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="e36ea-177">Yes (Spark NLP)</span></span> | <span data-ttu-id="e36ea-178">Igen (szövegelemzési API-val)</span><span class="sxs-lookup"><span data-stu-id="e36ea-178">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="e36ea-179">Nyelvfelismerés</span><span class="sxs-lookup"><span data-stu-id="e36ea-179">Language detection</span></span> | <span data-ttu-id="e36ea-180">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-180">No</span></span> | <span data-ttu-id="e36ea-181">Igen (szövegelemzési API-val)</span><span class="sxs-lookup"><span data-stu-id="e36ea-181">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="e36ea-182">Angol mellett több nyelvet is támogat</span><span class="sxs-lookup"><span data-stu-id="e36ea-182">Supports multiple languages besides English</span></span> | <span data-ttu-id="e36ea-183">Nem</span><span class="sxs-lookup"><span data-stu-id="e36ea-183">No</span></span> | <span data-ttu-id="e36ea-184">Igen (eltérő API-t)</span><span class="sxs-lookup"><span data-stu-id="e36ea-184">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="e36ea-185">Lásd még</span><span class="sxs-lookup"><span data-stu-id="e36ea-185">See also</span></span>

[<span data-ttu-id="e36ea-186">Természetes nyelvű feldolgozása</span><span class="sxs-lookup"><span data-stu-id="e36ea-186">Natural language processing</span></span>](../scenarios/natural-language-processing.md)
