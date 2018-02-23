---
title: "Выбор технологии приема сообщений в реальном времени"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 4f76e63a50c1d689ea3a37219a44aa94477a2e2e
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a><span data-ttu-id="07394-102">Выбор технологии приема сообщений в реальном времени в Azure</span><span class="sxs-lookup"><span data-stu-id="07394-102">Choosing a real-time message ingestion technology in Azure</span></span>

<span data-ttu-id="07394-103">Под обработкой в реальном времени подразумевается запись потоков данных в режиме реального времени и их обработка с минимальной задержкой.</span><span class="sxs-lookup"><span data-stu-id="07394-103">Real time processing deals with streams of data that are captured in real-time and processed with minimal latency.</span></span> <span data-ttu-id="07394-104">Для приема сообщений многим решениям для обработки в реальном времени требуется хранилище, которое можно использовать в качестве буфера. Такое хранилище должно поддерживать обработку с горизонтальным масштабированием, надежную доставку и другую семантику очереди сообщений.</span><span class="sxs-lookup"><span data-stu-id="07394-104">Many real-time processing solutions need a message ingestion store to act as a buffer for messages, and to support scale-out processing, reliable delivery, and other message queuing semantics.</span></span> 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a><span data-ttu-id="07394-105">Варианты при выборе технологии приема сообщений в реальном времени</span><span class="sxs-lookup"><span data-stu-id="07394-105">What are your options for real-time message ingestion?</span></span>

- [<span data-ttu-id="07394-106">Концентраторы событий Azure</span><span class="sxs-lookup"><span data-stu-id="07394-106">Azure Event Hubs</span></span>](/azure/event-hubs/)
- [<span data-ttu-id="07394-107">Центр Интернета вещей Azure</span><span class="sxs-lookup"><span data-stu-id="07394-107">Azure IoT Hub</span></span>](/azure/iot-hub/)
- <span data-ttu-id="07394-108">[Kafka в HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started).</span><span class="sxs-lookup"><span data-stu-id="07394-108">[Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)</span></span>

## <a name="azure-event-hubs"></a><span data-ttu-id="07394-109">Концентраторы событий Azure</span><span class="sxs-lookup"><span data-stu-id="07394-109">Azure Event Hubs</span></span>

<span data-ttu-id="07394-110">[Концентраторы событий Azure](/azure/event-hubs/) — это высокомасштабируемая платформа потоковой передачи данных и служба приема событий, принимающая и обрабатывающая миллионы событий в секунду.</span><span class="sxs-lookup"><span data-stu-id="07394-110">[Azure Event Hubs](/azure/event-hubs/) is a highly scalable data streaming platform and event ingestion service, capable of receiving and processing millions of events per second.</span></span> <span data-ttu-id="07394-111">Концентраторы событий могут обрабатывать и сохранять события, данные и телеметрию, созданные распределенным программным обеспечением и устройствами.</span><span class="sxs-lookup"><span data-stu-id="07394-111">Event Hubs can process and store events, data, or telemetry produced by distributed software and devices.</span></span> <span data-ttu-id="07394-112">Данные, отправляемые в концентратор событий, можно преобразовывать и сохранять с помощью любого поставщика аналитики в реальном времени, а также с помощью адаптеров пакетной обработки или хранения.</span><span class="sxs-lookup"><span data-stu-id="07394-112">Data sent to an event hub can be transformed and stored using any real-time analytics provider or batching/storage adapters.</span></span> <span data-ttu-id="07394-113">Концентраторы событий предоставляют возможности публикации и подписки с низкой задержкой в большом масштабе, что делает их подходящими для сценариев работы с большими данными.</span><span class="sxs-lookup"><span data-stu-id="07394-113">Event Hubs provides publish-subscribe capabilities with low latency at massive scale, which makes it appropriate for big data scenarios.</span></span>

## <a name="azure-iot-hub"></a><span data-ttu-id="07394-114">Центр Интернета вещей Azure</span><span class="sxs-lookup"><span data-stu-id="07394-114">Azure IoT Hub</span></span>

<span data-ttu-id="07394-115">[Центр Интернета вещей Azure](/azure/iot-hub/) — это управляемая служба, которая обеспечивает надежный и защищенный двунаправленный обмен данными между миллионами устройств Интернета вещей и облачным сервером.</span><span class="sxs-lookup"><span data-stu-id="07394-115">[Azure IoT Hub](/azure/iot-hub/) is a managed service that enables reliable and secure bidirectional communications between millions of IoT devices and a cloud-based back end.</span></span>

<span data-ttu-id="07394-116">Функции Центра Интернета вещей:</span><span class="sxs-lookup"><span data-stu-id="07394-116">Feature of IoT Hub include:</span></span>

* <span data-ttu-id="07394-117">несколько вариантов взаимодействия между устройствами и облаком,</span><span class="sxs-lookup"><span data-stu-id="07394-117">Multiple options for device-to-cloud and cloud-to-device communication.</span></span> <span data-ttu-id="07394-118">в частности одностороннюю передачу сообщений, передачу файлов и методы типа "запрос — ответ";</span><span class="sxs-lookup"><span data-stu-id="07394-118">These options include one-way messaging, file transfer, and request-reply methods.</span></span>
* <span data-ttu-id="07394-119">перенаправление сообщений в другие службы Azure;</span><span class="sxs-lookup"><span data-stu-id="07394-119">Message routing to other Azure services.</span></span>
* <span data-ttu-id="07394-120">хранилище для метаданных устройств и информации о состоянии синхронизации с возможностью запрашивания данных;</span><span class="sxs-lookup"><span data-stu-id="07394-120">Queryable store for device metadata and synchronized state information.</span></span>
* <span data-ttu-id="07394-121">безопасная связь и управление доступом с использованием отдельных ключей безопасности или сертификатов X.509 для каждого устройства;</span><span class="sxs-lookup"><span data-stu-id="07394-121">Scure communications and access control using per-device security keys or X.509 certificates.</span></span>
* <span data-ttu-id="07394-122">мониторинг событий управления взаимодействием и удостоверениями устройств.</span><span class="sxs-lookup"><span data-stu-id="07394-122">Monitoring of device connectivity and device identity management events.</span></span>

<span data-ttu-id="07394-123">С точки зрения приема сообщений функции Центра Интернета вещей и концентраторов событий похожи.</span><span class="sxs-lookup"><span data-stu-id="07394-123">In terms of message ingestion, IoT Hub is similar to Event Hubs.</span></span> <span data-ttu-id="07394-124">Но Центр Интернета вещей специально разработан для управления взаимодействием между устройствами Интернета вещей, а не только для приема сообщений.</span><span class="sxs-lookup"><span data-stu-id="07394-124">However, it was specifically designed for managing IoT device connectivity, not just message ingestion.</span></span> <span data-ttu-id="07394-125">Дополнительные сведения см. в статье [Сравнение Центра Интернета вещей Azure и концентраторов событий Azure](/azure/iot-hub/iot-hub-compare-event-hubs).</span><span class="sxs-lookup"><span data-stu-id="07394-125">For more information, see [Comparison of Azure IoT Hub and Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).</span></span> 

## <a name="kafka-on-hdinsight"></a><span data-ttu-id="07394-126">Kafka HDInsight</span><span class="sxs-lookup"><span data-stu-id="07394-126">Kafka on HDInsight</span></span>

<span data-ttu-id="07394-127">[Apache Kafka](https://kafka.apache.org/) — это распределенная платформа потоковой передачи с открытым кодом, которую можно использовать для создания приложений потоковой передачи и конвейеров данных в режиме реального времени.</span><span class="sxs-lookup"><span data-stu-id="07394-127">[Apache Kafka](https://kafka.apache.org/) is an open-source distributed streaming platform that can be used to build real-time data pipelines and streaming applications.</span></span> <span data-ttu-id="07394-128">Kafka также предоставляет функцию брокера сообщений, подобную очереди сообщений, с помощью которой можно выполнять публикацию и подписываться на именованные потоки данных.</span><span class="sxs-lookup"><span data-stu-id="07394-128">Kafka also provides message broker functionality similar to a message queue, where you can publish and subscribe to named data streams.</span></span> <span data-ttu-id="07394-129">Эта платформа поддерживает горизонтальное масштабирование, отличается отказоустойчивостью и высокой скоростью работы.</span><span class="sxs-lookup"><span data-stu-id="07394-129">It is horizontally scalable, fault-tolerant, and extremely fast.</span></span> <span data-ttu-id="07394-130">[Kafka в HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) предоставляет управляемую высокомасштабируемую службу с высоким уровнем доступности в Azure.</span><span class="sxs-lookup"><span data-stu-id="07394-130">[Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) provides a Kafka as a managed, highly scalable, and highly available service in Azure.</span></span> 

<span data-ttu-id="07394-131">Наиболее распространенные варианты использования Kafka:</span><span class="sxs-lookup"><span data-stu-id="07394-131">Some common use cases for Kafka are:</span></span>

* <span data-ttu-id="07394-132">**Обмен сообщениями**.</span><span class="sxs-lookup"><span data-stu-id="07394-132">**Messaging**.</span></span> <span data-ttu-id="07394-133">Благодаря поддержке модели обмена сообщениями по схеме "публикация — подписка" Kafka часто используется в качестве брокера сообщений.</span><span class="sxs-lookup"><span data-stu-id="07394-133">Because it supports the publish-subscribe message pattern, Kafka is often used as a message broker.</span></span>
* <span data-ttu-id="07394-134">**Отслеживание действий**.</span><span class="sxs-lookup"><span data-stu-id="07394-134">**Activity tracking**.</span></span> <span data-ttu-id="07394-135">Благодаря возможности ведения журнала записей в определенном порядке Kafka можно использовать для отслеживания и воссоздания действий, например действий пользователя на веб-сайте.</span><span class="sxs-lookup"><span data-stu-id="07394-135">Because Kafka provides in-order logging of records, it can be used to track and re-create activities, such as user actions on a web site.</span></span>
* <span data-ttu-id="07394-136">**Агрегирование**.</span><span class="sxs-lookup"><span data-stu-id="07394-136">**Aggregation**.</span></span> <span data-ttu-id="07394-137">С помощью потоковой обработки можно объединить информацию из разных потоков для ее централизации в качестве оперативных данных.</span><span class="sxs-lookup"><span data-stu-id="07394-137">Using stream processing, you can aggregate information from different streams to combine and centralize the information into operational data.</span></span>
* <span data-ttu-id="07394-138">**Преобразование**.</span><span class="sxs-lookup"><span data-stu-id="07394-138">**Transformation**.</span></span> <span data-ttu-id="07394-139">С помощью потоковой обработки можно объединять и дополнять данные из нескольких входных разделов в один или несколько выходных разделов.</span><span class="sxs-lookup"><span data-stu-id="07394-139">Using stream processing, you can combine and enrich data from multiple input topics into one or more output topics.</span></span>

## <a name="key-selection-criteria"></a><span data-ttu-id="07394-140">Основные критерии выбора</span><span class="sxs-lookup"><span data-stu-id="07394-140">Key selection criteria</span></span>

<span data-ttu-id="07394-141">Чтобы ограничить количество вариантов, сначала ответьте на следующие вопросы:</span><span class="sxs-lookup"><span data-stu-id="07394-141">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="07394-142">Вам нужно двустороннее взаимодействие между устройствами Интернета вещей и Azure?</span><span class="sxs-lookup"><span data-stu-id="07394-142">Do you need two-way communication between your IoT devices and Azure?</span></span> <span data-ttu-id="07394-143">Если да, выберите Центр Интернета вещей.</span><span class="sxs-lookup"><span data-stu-id="07394-143">If so, choose IoT Hub.</span></span>

- <span data-ttu-id="07394-144">Вам нужно управлять доступом для отдельных устройств и иметь возможность отменить доступ к определенному устройству?</span><span class="sxs-lookup"><span data-stu-id="07394-144">Do you need to manage access for individual devices and be able to revoke access to a specific device?</span></span> <span data-ttu-id="07394-145">Если да, выберите Центр Интернета вещей.</span><span class="sxs-lookup"><span data-stu-id="07394-145">If yes, choose IoT Hub.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="07394-146">Матрица возможностей</span><span class="sxs-lookup"><span data-stu-id="07394-146">Capability matrix</span></span>

<span data-ttu-id="07394-147">В следующих таблицах перечислены основные различия в возможностях.</span><span class="sxs-lookup"><span data-stu-id="07394-147">The following tables summarize the key differences in capabilities.</span></span> 

| | <span data-ttu-id="07394-148">Центр Интернета вещей</span><span class="sxs-lookup"><span data-stu-id="07394-148">IoT Hub</span></span> | <span data-ttu-id="07394-149">Концентраторы событий</span><span class="sxs-lookup"><span data-stu-id="07394-149">Event Hubs</span></span> | <span data-ttu-id="07394-150">Kafka HDInsight</span><span class="sxs-lookup"><span data-stu-id="07394-150">Kafka on HDInsight</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="07394-151">обмен данными между облаком и устройством;</span><span class="sxs-lookup"><span data-stu-id="07394-151">Cloud-to-device communications</span></span> | <span data-ttu-id="07394-152">Yes</span><span class="sxs-lookup"><span data-stu-id="07394-152">Yes</span></span> | <span data-ttu-id="07394-153">Нет </span><span class="sxs-lookup"><span data-stu-id="07394-153">No</span></span> | <span data-ttu-id="07394-154">Нет </span><span class="sxs-lookup"><span data-stu-id="07394-154">No</span></span> |
| <span data-ttu-id="07394-155">Отправка файлов, инициированная устройством</span><span class="sxs-lookup"><span data-stu-id="07394-155">Device-initiated file upload</span></span> | <span data-ttu-id="07394-156">Yes</span><span class="sxs-lookup"><span data-stu-id="07394-156">Yes</span></span> | <span data-ttu-id="07394-157">Нет </span><span class="sxs-lookup"><span data-stu-id="07394-157">No</span></span> | <span data-ttu-id="07394-158">Нет </span><span class="sxs-lookup"><span data-stu-id="07394-158">No</span></span> |
| <span data-ttu-id="07394-159">Сведения о состоянии устройства</span><span class="sxs-lookup"><span data-stu-id="07394-159">Device state information</span></span> | [<span data-ttu-id="07394-160">Двойники устройств</span><span class="sxs-lookup"><span data-stu-id="07394-160">Device twins</span></span>](/azure/iot-hub/iot-hub-devguide-device-twins) | <span data-ttu-id="07394-161">Нет </span><span class="sxs-lookup"><span data-stu-id="07394-161">No</span></span> | <span data-ttu-id="07394-162">Нет </span><span class="sxs-lookup"><span data-stu-id="07394-162">No</span></span> |
| <span data-ttu-id="07394-163">Поддержка протоколов</span><span class="sxs-lookup"><span data-stu-id="07394-163">Protocol support</span></span> | <span data-ttu-id="07394-164">MQTT, AMQP, HTTPS <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="07394-164">MQTT, AMQP, HTTPS <sup>1</sup></span></span> | <span data-ttu-id="07394-165">AMQP, HTTPS</span><span class="sxs-lookup"><span data-stu-id="07394-165">AMQP, HTTPS</span></span> | [<span data-ttu-id="07394-166">Протокол Kafka</span><span class="sxs-lookup"><span data-stu-id="07394-166">Kafka Protocol</span></span>](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| <span data-ttu-id="07394-167">Безопасность</span><span class="sxs-lookup"><span data-stu-id="07394-167">Security</span></span> | <span data-ttu-id="07394-168">Идентификация каждого устройства и управление доступом с возможностью его отмены.</span><span class="sxs-lookup"><span data-stu-id="07394-168">Per-device identity; revocable access control.</span></span> | <span data-ttu-id="07394-169">Политики общего доступа с ограниченными возможностями отмены доступа с помощью политик издателя.</span><span class="sxs-lookup"><span data-stu-id="07394-169">Shared access policies; limited revocation through publisher policies.</span></span> | <span data-ttu-id="07394-170">Поддержка аутентификации с использованием SASL, модульной авторизации, интеграция с внешними службами аутентификации.</span><span class="sxs-lookup"><span data-stu-id="07394-170">Authentication using SASL; pluggable authorization; integration with external authentication services supported.</span></span> |

<span data-ttu-id="07394-171">[1] [Шлюз протокола Интернета вещей Azure](/azure/iot-hub/iot-hub-protocol-gateway) можно также использовать как пользовательский шлюз для включения адаптации протокола для Центра Интернета вещей.</span><span class="sxs-lookup"><span data-stu-id="07394-171">[1] You can also use [Azure IoT protocol gateway](/azure/iot-hub/iot-hub-protocol-gateway) as a custom gateway to enable protocol adaptation for IoT Hub.</span></span>

<span data-ttu-id="07394-172">Дополнительные сведения см. в статье [Сравнение Центра Интернета вещей Azure и концентраторов событий Azure](/azure/iot-hub/iot-hub-compare-event-hubss).</span><span class="sxs-lookup"><span data-stu-id="07394-172">For more information, see [Comparison of Azure IoT Hub and Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubss).</span></span>