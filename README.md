# apps-svc

# Setup


Follow these steps to get started.

* Clone the following apps and push them to your docker repository on `localhost:5000`:
  * [github.com/direktiv-apps/image-magick](https://github.com/direktiv-apps/image-magick)
  * [github.com/direktiv-apps/swagger2markdown](https://github.com/direktiv-apps/swagger2markdown)
  * [github.com/direktiv-apps/usql](https://github.com/direktiv-apps/usql)
  * [github.com/direktiv-apps/yaml2json](https://github.com/direktiv-apps/yaml2json)
* Choose a postgres database server for the services to use. One option is to port-forward the same database server used by Direktiv.
* Create credentials and a database on the database server.
* Use Direktiv's git mirroring functionality to clone this repository into a namespace called `apps-svc`.
* In the `apps-svc` namespace, create three secrets to use the database:
  * dbaddr (example: `192.168.1.10:5432/apps`)
  * dbuser
  * dbpass
* In the `apps-svc` namespace, execute the `/initdb` workflow. Check that it returns successfully.

# Add Data

## Adding App

Execute the `/push` workflow with a JSON input containing the following fields:

```json
{
  "data": "c3dhZ2dlc...",
  "logo": "iVBORw0KG..."
}
```

The data field contains the swagger.yaml of that service. The logo field is optional and contains the logo for the application (for all tags).


## Upload Icon 

Optionally, you can upload a custom icon for the app. Icons are stored on a per-app level, which means all tags for the same app share the same icon file. 

To upload a custom icon, execute the `/upload-icon` workflow with a JSON input containing the following two fields:

```json
{
    "name": "myapp",
    "data": "SGVsbG8="
}
```

## Rebuilding the Service

In it's current architecture, changes made with the previous two workflows do have some impact immediately. But they don't propagate through the entire set of service APIs until the service has been "rebuilt". For this purpose there is the `/rebuild` workflow, which needs to be executed. 

Currently, the rebuild workflow needs to be manually executed. That's to give flexibility to the user. You could make it a daily CRON-triggered workflow, call it once for each and every change you make, or call it after a batch of changes have been applied all at once.

No input or arguments are required for this workflow.


# APIs

## Get Icon (PNG)

`GET /api/namespaces/apps-svc/tree/icon?op=wait&input.name={{name}}&field=icon&raw-output=true&ctype=image/png`

This API returns a icon-sized PNG file for the app identified by `{{name}}`. This API returns a placeholder icon even if none were defined and even if the app doesn't exist.

## Get List 

`GET /api/namespaces/apps-svc/tree/list-expanded?op=wait&field=output&raw-output=true&ctype=application/json`

This API returns an expanded list of all known apps and the latest tag, with a short description of what each does. It requires no arguments because it returns all results. If you receive abnormal data from this API something went wrong during the `rebuild` step.

## Get Detail 

`GET /api/namespaces/apps-svc/tree/detail?op=wait&input.name={{name}}&input.tag={{tag}}&raw-output=true&ctype=application/json`

This API returns detailed information about the application specified by `{{name}}` and `{{tag}}`. 


