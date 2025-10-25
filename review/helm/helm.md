- ### Deploying a Chart with Helm CLI
	- For this course: we have a specific configuration, so there is a fork of the postgres helm chart, which is available in [lfs269/helm-charts | helm-charts](https://lfs269.github.io/helm-charts/)
	- [helm-charts/charts/postgres at master · lfs269/helm-charts · GitHub](https://github.com/lfs269/helm-charts/tree/master/charts/postgres)
	- To setup helm to deploy a postgres chart from a repo, you need to first create a repo.
	- `helm repo list`: list repositories (right now, there's none)
	- `helm repo add [NAME] [URL] [FLAGS]`
	  `helm repo add lfs269 https://lfs269.github.io/helm-charts/`  
	- You would then validate with
	  `helm repo list`  
	- Next would be to fetch the index from this repository using
	  `helm repo update`  
		-
		  > what is updating the helm index after adding a helm repo #chatgpt  
			- Good question — this touches on how Helm repositories work internally.
			- Let’s break it down clearly:
			-
---
			- ### 🧭 Context
				- When you **add a Helm repository**, you’re telling Helm where to look for charts. For example:
				-
				  ```
				  helm repo add my-repo https://example.com/charts
				  ```
				- This command:
					- Adds the repo URL to your local Helm configuration (`~/.config/helm/repositories.yaml`).
					- Downloads the **index file** (`index.yaml`) from the repo URL and caches it locally.
				-
---
			- ### 📦 What is the Helm index?
				- Each Helm chart repository contains a file called **`index.yaml`**, typically found at:
				-
				  ```
				  https://example.com/charts/index.yaml
				  ```
				- This file lists **metadata about all the charts** in that repository — including:
					- Chart names
					- Versions
					- URLs to `.tgz` chart packages
					- Descriptions, dependencies, etc.
				- Think of it as the "catalog" for that repo.
				-
---
			- ### 🔄 What does “updating the Helm index” mean?
				- There are two slightly different meanings depending on *where* the update happens:
				- #### 1. **Client-side update (local cache refresh)**
					- When you run:
					-
					  ```
					  helm repo update
					  ```
					- Helm fetches the latest `index.yaml` from each repo you’ve added and updates your local cache in:
					-
					  ```
					  ~/.cache/helm/repository/
					  ```
					- This ensures your Helm client knows about the newest charts and versions available.
					- **So this is what happens "after adding a Helm repo" if you want to see updated charts.**
					-
---
				- #### 2. **Server-side update (repo maintainer side)**
					- If *you* are hosting a Helm repository, you might need to update the `index.yaml` file whenever you add or remove charts from it. This is done with:
					-
					  ```
					  helm repo index .
					  ```
					- That command generates or updates the `index.yaml` file based on the `.tgz` chart packages present in the current directory.
					- Example workflow for repo maintainers:
					-
					  ```
					  helm package mychart/
					  helm repo index . --url https://example.com/charts
					  ```
					-
---
			- ### ⚙️ Summary
				-
				  | Task | Command | Purpose |
				  |---|---|---|
				  | Add a repo | `helm repo add my-repo https://example.com/charts` | Registers repo and downloads initial index |
				  | Refresh local catalog | `helm repo update` | Updates cached `index.yaml` files |
				  | Create/maintain repo index (for maintainers) | `helm repo index .` | Builds or updates the `index.yaml` file listing charts in your repo |
				-
---
				- Would you like me to clarify **how to automate updating the Helm index when you add new charts** (for hosting your own repo)?
	- To find out a chart to install:
	  `$ helm search repo lfs269`  
	  `$ helm search repo postgres`  
	  Gives you: `lfs269/postgres`  
	- To check the releases:
	  `helm list [-n instavote]` (show no releases now)  
	- To deploy a chart (create a release):
	  `helm install [NAME] [CHART] [flags]` (NAME = release name)  
	  ```
	  helm install db lfs269/postgres \
	    --set settings.authMethod=trust,service.name=db \
	    --dry-run
	  ```
	  ```
	  helm install db lfs269/postgres \
	    --set settings.authMethod=trust,service.name=db
	  ```
		- When you deploy a chart it creates a release: a specific release version deployed in your environment
		- Some of the common options include providing kustomization values for the chart you want to deploy in your environment. You can look at them browsing to the chart and checking the documentation or `values.yaml`.
		- You can also add `--namespace=instavote` to the `helm install` options
	- To uninstall the release:
	  `helm uninstall [NAME]` (NAME = release name)  
	  `helm uninstall db`  
	- `k logs pod/worker-8499496f88-x2k9h`
		- Now you see that the worker is `Connected to db` and started to process the votes
