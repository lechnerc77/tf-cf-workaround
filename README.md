# Workaround for Automation of Terraform and CF

## Status Quo - Terraform and Cloud Foundry

Currently there are gaps in the Terraform Provider for Cloud Foundry especially when it comes to V3 API functionality and SAP BTP specifics

## Workaround

If you automate via Terraform and need to use V3 API functionality or SAP BTP specifics and GitHub Actions, you can use the following workaround provided in this repository:

- Providing a custom GitHub Action that leverages the latest CF CLI. The code is available in the directory `.github/actions/sap-btp-cf`
- Calling the custom Action in your GitHub Action and basically calling the CF CLI commands via the custom Action.

## Examples for GitHub Action Steps

Example for setting user roles on org and space level

```yaml
- name: Assign CF users
  id: assign_cf_users
  uses: ./.github/actions/sap-btp-cf
  with:
    cf_api: ${{ env.CF_API_URL }}
    cf_username: ${{ secrets.CF_USERNAME }}
    cf_password: ${{ secrets.CF_PASSWORD }}
    cf_org: ${{ env.CF_ORG_NAME }}
    cf_space: ${{ env.CF_SPACE_NAME }}
    command: |
      cf8 set-org-role user@sap.com ${{ env.CF_ORG_NAME }} OrgManager \
      && cf8 set-org-role user2@sap.com ${{ env.CF_ORG_NAME }} OrgManager --origin sap.ids\
      && cf8 set-space-role user1@sap.com ${{ env.CF_ORG_NAME }} ${{ env.CF_SPACE_NAME }} SpaceManager \
      && cf8 set-space-role user1@sap.com ${{ env.CF_ORG_NAME }} ${{ env.CF_SPACE_NAME }} SpaceDeveloper \
      && cf8 set-space-role user2@sap.com ${{ env.CF_ORG_NAME }} ${{ env.CF_SPACE_NAME }} SpaceManager  --origin sap.ids\
      && cf8 set-space-role user2@sap.com ${{ env.CF_ORG_NAME }} ${{ env.CF_SPACE_NAME }} SpaceDeveloper  --origin sap.ids
```

Example for deploying an MTAR file:

```yaml
- name: Deploy MTAR
  id: deploy_mtar
  uses: ./.github/actions/sap-btp-cf
  with:
    cf_api: ${{ env.CF_API_URL }}
    cf_username: ${{ secrets.BTP_USERNAME }}
    cf_password: ${{ secrets.BTP_PASSWORD }}
    cf_org: ${{ env.CF_ORG_NAME }}
    cf_space: ${{ env.CF_SPACE_NAME }}
    command: |
      cf8 install-plugin -f multiapps
      cf8 deploy ${{ env.MTAR_PATH }}/${{ env.MTAR_FILE }} -f
```

Example for installing the community plugin `html5-plugin` and fetching the app URL:

```yaml
- name: Fetch app details
  id: fetch_app_url
  uses: ./.github/actions/sap-btp-cf
  with:
    cf_api: ${{ env.CF_API_URL }}
    cf_username: ${{ secrets.BTP_USERNAME }}
    cf_password: ${{ secrets.BTP_PASSWORD }}
    cf_org: ${{ env.CF_ORG_NAME }}
    cf_space: ${{ env.CF_SPACE_NAME }}
    command: |
      cf8 install-plugin -r CF-Community "html5-plugin" -f
      echo "APP_URL_OUTPUT=$(cf html5-list -d -u | grep -Eo '(http|https)://[a-zA-Z0-9./?=_%:-]*' | grep -Eo '.*Launchpad.*' | sed -e 's/.cpp./.launchpad./g')" >> $GITHUB_ENV 
```

## Points to consider

If you have plugins that are always used in your automation, you can install them in the custom Action. This way you don't have to install them in every step of your automation.