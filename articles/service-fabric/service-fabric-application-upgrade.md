---
title: "Service Fabric 應用程式升級 | Microsoft Docs"
description: "本文章提供升級 Service Fabric 應用程式的簡介，其中包括選擇升級模式和執行健康狀態檢查。"
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: 
ms.assetid: 803c9c63-373a-4d6a-8ef2-ea97e16e88dd
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 8/9/2017
ms.author: subramar
ms.translationtype: Human Translation
ms.sourcegitcommit: b4637922e7b280b0e9954c9e51788202e784b4f9
ms.openlocfilehash: 743223f78f279fedf33f73ff52b56f4a7358cd51
ms.contentlocale: zh-tw
ms.lasthandoff: 02/13/2017

---
# <a name="service-fabric-application-upgrade"></a>Service Fabric 應用程式升級
Azure Service Fabric 應用程式是服務集合。 在升級期間，Service Fabric 會比較新的 [應用程式資訊清單](service-fabric-application-model.md#describe-an-application) 與舊版本，並決定應用程式中哪些服務需要更新。 Service Fabric 會比較服務資訊清單中的版本號碼和上一版中的版本號碼。 如果服務未變更，則該服務不會升級。

## <a name="rolling-upgrades-overview"></a>輪流升級概觀
在輪流應用程式升級中，升級是階段執行。 在每個階段中，升級會套用至叢集中的節點子集，稱為更新網域。 如此一來，應用程式在整個升級過程中仍然可供使用。 升級期間，叢集可能包含舊和新版本的混合。

因此，兩個版本必須是向前及向後相容。 如果它們不相容，應用程式系統管理員負責執行多階段升級，以維護可用性。 在多階段升級中，第一個步驟是升級到與先前版本相容的應用程式中繼版本。 第二個步驟是升級與更新前版本中斷相容性，但與中繼版本相容的最終版本。

當您設定叢集時，會在叢集資訊清單中指定更新網域。 更新網域不會以特定順序收到更新。 更新網域是應用程式的部署邏輯單元。 更新網域可以讓服務在升級期間維持高可用性。

如果升級套用到叢集中的所有節點 (應用程式只有一個更新網域就是這種情形)，則不可能進行輪流升級。 不建議使用此方法，因為升級時服務會中斷且無法使用。 此外，當叢集僅以一個更新網域進行設定時，Azure 不提供任何保證。

## <a name="health-checks-during-upgrades"></a>升級期間的健康狀態檢查
對於升級，需要設定健康狀態原則 (或可能會使用預設值)。 當所有更新網域在指定的逾時內升級，且所有更新網域都視為健康時，則稱為成功升級。  健康的更新網域意指更新網域會通過健康狀態原則中指定的所有健康狀態檢查。 例如，健康狀態原則可能會要求應用程式執行個體中的所有服務都必須 *健康狀態良好*，如 Service Fabric 定義的健康狀況。

Service Fabric 在升級期間進行的健康狀態原則以及檢查不限於服務和應用程式。 也就是說，不會完成任何服務專屬的測試。  例如，您的服務可能需要輸送量，但 Service Fabric 並沒有檢查輸送量的資訊。 如需了解會執行的檢查，請參閱 [健康狀態文章](service-fabric-health-introduction.md) 。 在升級期間進行的檢查包括是否已正確地複製應用程式封裝、是否已啟動執行個體等等。

應用程式健康狀態是應用程式的子系實體彙總。 簡單地說，Service Fabric 會透過應用程式報告的健康狀態來評估應用程式的健康狀態。 它也會以此種方式評估應用程式所有服務的健康狀態。 Service Fabric 會進一步藉由彙總其子項的健康狀態 (例如服務複本) 來評估應用程式服務的健康狀態。 一旦符合應用程式健康狀態原則，便可以繼續升級。 如果違反健康狀態原則，則應用程式升級將會失敗。

## <a name="upgrade-modes"></a>升級模式
我們建議的應用程式升級模式為監視模式，這是常用的模式。 監視模式會在一個更新網域上執行升級，如果所有健康狀態檢查都通過 (每個指定的原則)，即會自動移至下一個更新網域。  如果健康狀態檢查失敗及/或達到逾時，更新網域的升級就會回復，或者手動變更為未受監視的手動模式。 您可以設定升級在升級失敗時選擇這兩種其中一種模式。 

不受監控手動模式在每次於更新網域上升級之後都需要手動介入，以開始進行下一個更新網域上的升級。 系統不會執行任何 Service Fabric 健康狀態檢查。 系統管理員在開始下一個更新網域中的升級之前，會執行健康狀態或狀態檢查。

## <a name="upgrade-default-services"></a>升級預設服務
您可以在應用程式的升級程序期間升級 Service Fabric 應用程式內的預設服務。 預設服務會在[應用程式資訊清單](service-fabric-application-model.md#describe-an-application)中定義。 升級預設服務的標準規則如下︰

1. 會建立不存在於叢集中的新[應用程式資訊清單](service-fabric-application-model.md#describe-an-application)中的預設服務。
> [!TIP]
> [EnableDefaultServicesUpgrade](service-fabric-cluster-fabric-settings.md#fabric-settings-that-you-can-customize) 必須設為 true，以啟用下列規則。 從 v5.5 可支援此功能。

2. 會更新在上一個[應用程式資訊清單](service-fabric-application-model.md#describe-an-application)及新版本中的預設服務。 在新版本中的服務描述將會覆寫已在叢集中的部分。 在更新預設服務失敗時，應用程式升級會自動回復。
3. 會刪除上一個[應用程式資訊清單](service-fabric-application-model.md#describe-an-application)中的預設服務，但不會刪除新版中的預設服務。 **請注意，無法還原此刪除預設服務。**

如果應用程式升級已回復，預設服務會還原為開始升級前的狀態。 但永遠無法建立已刪除的服務。

## <a name="application-upgrade-flowchart"></a>應用程式升級流程圖
本段落下面的流程圖可以協助您了解 Service Fabric 應用程式的升級程序。 特別是，此流程會說明逾時 (包括 HealthCheckStableDuration、HealthCheckRetryTimeout 和 UpgradeHealthCheckInterval) 如何協助控制一個更新網域中的升級被視為成功或失敗的時機。

![Service Fabric 應用程式的升級程序][image]

## <a name="next-steps"></a>後續步驟
[使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md) 將引導您完成使用 Visual Studio 進行應用程式升級的步驟。

[使用 PowerShell 升級您的應用程式](service-fabric-application-upgrade-tutorial-powershell.md) 將引導您完成使用 PowerShell 進行應用程式升級的步驟。

使用 [升級參數](service-fabric-application-upgrade-parameters.md)來控制您應用程式的升級方式。

了解如何使用 [資料序列化](service-fabric-application-upgrade-data-serialization.md)，以讓您的應用程式升級相容。

參考 [進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。

參考 [疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。

[image]: media/service-fabric-application-upgrade/service-fabric-application-upgrade-flowchart.png

