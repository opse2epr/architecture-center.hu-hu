---
title: A természetes nyelvi feldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 20dc51e661befcc09dd1751b031d445ff2b9fa1a
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486488"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="264b5-102">Az Azure-ban technológia feldolgozása természetes nyelv kiválasztása</span><span class="sxs-lookup"><span data-stu-id="264b5-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="264b5-103">Szabad formátumú szöveges feldolgozási hajtja végre az szöveg-, keresési, támogatása céljából általában bekezdéseit tartalmazó dokumentumok, de is szolgál, mint a hangulatelemzés, témafelismerés, természetes nyelvi feldolgozást (NLP) feladatait más nyelv észlelése, a kulcsszókeresést és kategorizálási dokumentum.</span><span class="sxs-lookup"><span data-stu-id="264b5-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="264b5-104">Ez a cikk a technológiai lehetőségekhez támogatásához a NLP feladatokat-kiszolgálóként működő összpontosít.</span><span class="sxs-lookup"><span data-stu-id="264b5-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="264b5-105">Mik azok a beállítások egy NLP szolgáltatás kiválasztásakor?</span><span class="sxs-lookup"><span data-stu-id="264b5-105">What are your options when choosing an NLP service?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="264b5-106">Az Azure-ban a következő szolgáltatásokat nyújtanak a természetes nyelvi feldolgozási (NLP) képességek:</span><span class="sxs-lookup"><span data-stu-id="264b5-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="264b5-107">Az Azure HDInsight Spark-és Spark MLlib</span><span class="sxs-lookup"><span data-stu-id="264b5-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="264b5-108">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="264b5-108">Azure Databricks</span></span>](/azure/azure-databricks/what-is-azure-databricks)
- [<span data-ttu-id="264b5-109">A Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="264b5-109">Microsoft Cognitive Services</span></span>](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a><span data-ttu-id="264b5-110">Fontosabb kritériumok</span><span class="sxs-lookup"><span data-stu-id="264b5-110">Key selection criteria</span></span>

<span data-ttu-id="264b5-111">Így szűkítheti, első lépésként a kérdések megválaszolása:</span><span class="sxs-lookup"><span data-stu-id="264b5-111">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="264b5-112">Biztosan előre összeállított modellek használata?</span><span class="sxs-lookup"><span data-stu-id="264b5-112">Do you want to use prebuilt models?</span></span> <span data-ttu-id="264b5-113">Ha igen, érdemes a Microsoft Cognitive Services által kínált az API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="264b5-113">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="264b5-114">Szükség van egy nagy forrásgyűjteményébe álló szöveges adatok elleni egyéni modelleket taníthat be?</span><span class="sxs-lookup"><span data-stu-id="264b5-114">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="264b5-115">Ha igen, fontolja meg az Azure HDInsight Spark MLlib és a Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="264b5-115">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="264b5-116">A jogkivonatok, származtató, morfológiai és kifejezés gyakoriság/inverz dokumentum gyakorisága (TF/IDF) alacsony szintű NLP képességek szükségesek?</span><span class="sxs-lookup"><span data-stu-id="264b5-116">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="264b5-117">Ha igen, fontolja meg az Azure HDInsight Spark MLlib és a Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="264b5-117">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="264b5-118">Egyszerű, magas szintű NLP szolgáltatásokat, például az entitás- és leképezés azonosítása, témafelismerés, helyesírás-ellenőrzés vagy hangulatelemzés kell?</span><span class="sxs-lookup"><span data-stu-id="264b5-118">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="264b5-119">Ha igen, érdemes a Microsoft Cognitive Services által kínált az API-k használatával.</span><span class="sxs-lookup"><span data-stu-id="264b5-119">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="264b5-120">Képességmátrix</span><span class="sxs-lookup"><span data-stu-id="264b5-120">Capability matrix</span></span>

<span data-ttu-id="264b5-121">A következő táblázat összefoglalja a fő különbségeket, a képességek.</span><span class="sxs-lookup"><span data-stu-id="264b5-121">The following tables summarize the key differences in capabilities.</span></span>

### <a name="general-capabilities"></a><span data-ttu-id="264b5-122">Általános képességek</span><span class="sxs-lookup"><span data-stu-id="264b5-122">General capabilities</span></span>

| | <span data-ttu-id="264b5-123">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="264b5-123">Azure HDInsight</span></span> | <span data-ttu-id="264b5-124">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="264b5-124">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="264b5-125">Modellek imagenet kínál szolgáltatásként</span><span class="sxs-lookup"><span data-stu-id="264b5-125">Provides pretrained models as a service</span></span> | <span data-ttu-id="264b5-126">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-126">No</span></span> | <span data-ttu-id="264b5-127">Igen</span><span class="sxs-lookup"><span data-stu-id="264b5-127">Yes</span></span> |
| <span data-ttu-id="264b5-128">REST API</span><span class="sxs-lookup"><span data-stu-id="264b5-128">REST API</span></span> | <span data-ttu-id="264b5-129">Igen</span><span class="sxs-lookup"><span data-stu-id="264b5-129">Yes</span></span> | <span data-ttu-id="264b5-130">Igen</span><span class="sxs-lookup"><span data-stu-id="264b5-130">Yes</span></span> |
| <span data-ttu-id="264b5-131">Programozhatóság</span><span class="sxs-lookup"><span data-stu-id="264b5-131">Programmability</span></span> | <span data-ttu-id="264b5-132">Python, Scala, Java</span><span class="sxs-lookup"><span data-stu-id="264b5-132">Python, Scala, Java</span></span> | <span data-ttu-id="264b5-133">C#, Java, Node.js, Python, PHP, Ruby</span><span class="sxs-lookup"><span data-stu-id="264b5-133">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="264b5-134">Big data jellegű adatkészletek és a nagy dokumentumok feldolgozásának támogatásához</span><span class="sxs-lookup"><span data-stu-id="264b5-134">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="264b5-135">Igen</span><span class="sxs-lookup"><span data-stu-id="264b5-135">Yes</span></span> | <span data-ttu-id="264b5-136">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-136">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="264b5-137">Az alsó szintű természetes nyelvi feldolgozási képességek</span><span class="sxs-lookup"><span data-stu-id="264b5-137">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="264b5-138">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="264b5-138">Azure HDInsight</span></span> | <span data-ttu-id="264b5-139">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="264b5-139">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- |
| <span data-ttu-id="264b5-140">Jogkivonatokat létrehozó</span><span class="sxs-lookup"><span data-stu-id="264b5-140">Tokenizer</span></span> | <span data-ttu-id="264b5-141">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-141">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-142">Igen (nyelvi elemzési API-t)</span><span class="sxs-lookup"><span data-stu-id="264b5-142">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="264b5-143">Szótőkereső</span><span class="sxs-lookup"><span data-stu-id="264b5-143">Stemmer</span></span> | <span data-ttu-id="264b5-144">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-144">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-145">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-145">No</span></span> |
| <span data-ttu-id="264b5-146">Lemmatizer</span><span class="sxs-lookup"><span data-stu-id="264b5-146">Lemmatizer</span></span> | <span data-ttu-id="264b5-147">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-147">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-148">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-148">No</span></span> |
| <span data-ttu-id="264b5-149">Szövegrészeket része</span><span class="sxs-lookup"><span data-stu-id="264b5-149">Part of speech tagging</span></span> | <span data-ttu-id="264b5-150">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-150">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-151">Igen (nyelvi elemzési API-t)</span><span class="sxs-lookup"><span data-stu-id="264b5-151">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="264b5-152">Kifejezés gyakoriság/inverz-dokumentum gyakorisága (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="264b5-152">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="264b5-153">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="264b5-153">Yes (Spark MLlib)</span></span> | <span data-ttu-id="264b5-154">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-154">No</span></span> |
| <span data-ttu-id="264b5-155">Karakterlánc-hasonlóság&mdash;távolság számítási szerkesztése</span><span class="sxs-lookup"><span data-stu-id="264b5-155">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="264b5-156">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="264b5-156">Yes (Spark MLlib)</span></span> | <span data-ttu-id="264b5-157">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-157">No</span></span> |
| <span data-ttu-id="264b5-158">N-Gram típusú számítási</span><span class="sxs-lookup"><span data-stu-id="264b5-158">N-gram calculation</span></span> | <span data-ttu-id="264b5-159">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="264b5-159">Yes (Spark MLlib)</span></span> | <span data-ttu-id="264b5-160">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-160">No</span></span> |
| <span data-ttu-id="264b5-161">Állítsa le a word eltávolítása</span><span class="sxs-lookup"><span data-stu-id="264b5-161">Stop word removal</span></span> | <span data-ttu-id="264b5-162">Igen (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="264b5-162">Yes (Spark MLlib)</span></span> | <span data-ttu-id="264b5-163">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-163">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="264b5-164">Magas szintű természetes nyelvi feldolgozási képességek</span><span class="sxs-lookup"><span data-stu-id="264b5-164">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="264b5-165">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="264b5-165">Azure HDInsight</span></span> | <span data-ttu-id="264b5-166">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="264b5-166">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="264b5-167">Entitás/szándékot azonosítása és kinyerése</span><span class="sxs-lookup"><span data-stu-id="264b5-167">Entity/intent identification & extraction</span></span> | <span data-ttu-id="264b5-168">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-168">No</span></span> | <span data-ttu-id="264b5-169">Igen (Language Understanding Intelligent Service (LUIS) API)</span><span class="sxs-lookup"><span data-stu-id="264b5-169">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |
| <span data-ttu-id="264b5-170">Téma azonosítása</span><span class="sxs-lookup"><span data-stu-id="264b5-170">Topic detection</span></span> | <span data-ttu-id="264b5-171">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-171">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-172">Igen (szövegelemzési API-val)</span><span class="sxs-lookup"><span data-stu-id="264b5-172">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="264b5-173">Helyesírás-ellenőrzés</span><span class="sxs-lookup"><span data-stu-id="264b5-173">Spell checking</span></span> | <span data-ttu-id="264b5-174">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-174">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-175">Igen (a Bing Spell Check API bemutatása)</span><span class="sxs-lookup"><span data-stu-id="264b5-175">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="264b5-176">Hangulatelemzés</span><span class="sxs-lookup"><span data-stu-id="264b5-176">Sentiment analysis</span></span> | <span data-ttu-id="264b5-177">Igen (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="264b5-177">Yes (Spark NLP)</span></span> | <span data-ttu-id="264b5-178">Igen (szövegelemzési API-val)</span><span class="sxs-lookup"><span data-stu-id="264b5-178">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="264b5-179">Nyelvfelismerés</span><span class="sxs-lookup"><span data-stu-id="264b5-179">Language detection</span></span> | <span data-ttu-id="264b5-180">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-180">No</span></span> | <span data-ttu-id="264b5-181">Igen (szövegelemzési API-val)</span><span class="sxs-lookup"><span data-stu-id="264b5-181">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="264b5-182">Angol mellett több nyelvet is támogat</span><span class="sxs-lookup"><span data-stu-id="264b5-182">Supports multiple languages besides English</span></span> | <span data-ttu-id="264b5-183">Nincs</span><span class="sxs-lookup"><span data-stu-id="264b5-183">No</span></span> | <span data-ttu-id="264b5-184">Igen (eltérő API-t)</span><span class="sxs-lookup"><span data-stu-id="264b5-184">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="264b5-185">Lásd még</span><span class="sxs-lookup"><span data-stu-id="264b5-185">See also</span></span>

[<span data-ttu-id="264b5-186">Természetes nyelvű feldolgozása</span><span class="sxs-lookup"><span data-stu-id="264b5-186">Natural language processing</span></span>](../scenarios/natural-language-processing.md)
