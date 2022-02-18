# GitOps

```mermaid
flowchart LR
  dev(("Developer"))
  subgraph SCM ["Git"]
    direction TB
    prj[Project sources]
    iac[Helm charts]
    hr[Helm releases]
  end
  dev --> prj
  dev --> iac
  dev --> hr
  subgraph CI ["CI pipeline"]
    direction TB
    subgraph appci ["Applications CI"]
      direction LR
      build[Build] --> test[Tests]
      test --> upload["Upload package"]
      upload --> image["Build/upload container image"]
    end
    subgraph helmci ["Helm CI"]
       pack["Package/upload"]
    end
  end
  prj -->|Commit triggers| appci
  iac -->|Commit triggers| helmci
  subgraph ART ["Artifacts repository"]
    direction TB
    packagesrepo["language-dependent Packages"]
    containerrepo["Container images"]
    helmrepo["Helm charts"]
  end
  subgraph ART ["Artifacts repository"]
    direction TB
    packagesrepo["language-dependent Packages"]
    containerrepo["Container images"]
    helmrepo["Helm charts"]
  end
  upload -->|push built package| packagesrepo
  build -->|pull dependencies| packagesrepo
  image -->|push container image| containerrepo
  pack --> helmrepo
  hr -..-|references| helmrepo
  subgraph CD ["CD pipeline"]
    direction TB
    orchauto["Automation"] -..->|lookup release| containerrepo
    orchauto -->|push updated Helm release| hr
    orchupdate["Updater"] -->|pull Helm release| hr
  end
  hr --> |Commit triggers| orchupdate
  subgraph cluster ["Kubernetes Cluster"]
    direction TB
    orchupdate -->|push Helm release| helmop["Helm operator"]
    helmop -->|pull Helm chart| helmrepo
    helmop -->|deploy| pod["Target Pod"]
    pod --->|pull image| containerrepo
  end

```
