# apps-svc

# Setup

Follow these steps to get started.

* Clone the following apps and push them to your docker repository on `localhost:5000`:
  * github.com/direktiv-apps/image-magick
  * github.com/direktiv-apps/swagger2markdown
  * github.com/direktiv-apps/usql
  * github.com/direktiv-apps/yaml2json
* Choose a postgres database server for the services to use. One option is to port-forward the same database server used by Direktiv.
* Create credentials and a database on the database server.
* Use Direktiv's git mirroring functionality to clone this repository into a namespace called `apps-svc`.
* In the `apps-svc` namespace, create three secrets to use the database:
  * dbaddr (example: `192.168.1.10:5432/apps`)
  * dbuser
  * dbpass
* In the `apps-svc` namespace, execute the `/initdb` workflow. Check that it returns successfully.

Consider importing `insomnia.json` into Insomnia for easy access to working requests.

# Adding Data

Information about apps needs to be pushed into the services manually. 

## Upload App

Execute the `/push` workflow with a JSON input containing the following three fields:

```json
{
    "uri": "direktiv/helloworld",
    "tag": "latest",
    "data": "SGVsbG8="
}
```

The `uri` and `tag` fields match the docker container fields of the same name. The `data` field should have a base64 encoded swagger.yaml file.

If the URI doesn't include a domain it is presumed to be `docker.io`. The `latest` tag is not handled automatically: if you want to upload information about an app and give it multiple tags (example: `v1.0` & `latest`) you must execute this workflow twice, once with each tag.

## Upload Icon 

Optionally, you can upload a custom icon for the app. Icons are stored on a per-`uri` level, which means all tags for the same uri share the same icon file. 

To upload a custom icon, execute the `/upload-icon` workflow with a JSON input containing the following two fields:

```json
{
    "uri": "direktiv/helloworld",
    "data": "SGVsbG8="
}
```

In this case, `data` should be a base64 encoded PNG file. The dimensions will be automatically adjusted by the services. 

## Rebuilding the Service

In it's current prototype architecture, changes made with the previous two workflows do have some impact immediately. But they don't propagate through the entire set of service APIs until the service has been "rebuilt". For this purpose there is the `/rebuild` workflow, which needs to be executed. 

Currently, the rebuild workflow needs to be manually executed. That's to give flexibility to the user. You could make it a daily CRON-triggered workflow, call it once for each and every change you make, or call it after a batch of changes have been applied all at once.

No input or arguments are required for this workflow.

# Editing / Deleting Data

Editing data is as simple as pretending that the old data doesn't exist. Use the same APIs from the previous section to overwrite existing information.

Deleting data is a work-in-progress. You may notice that there are some workflows that seem like they should do this, but they aren't finished. For now, if you really need to delete any data you should manually delete the `apps` table in your database and wipe the `apps-svc` namespace, and start again. 

# APIs

## Get Icon (PNG)

`GET /api/namespaces/apps-svc/tree/icon?op=wait&input.uri={{uri}}&field=output&raw-output=true&ctype=image/png`

This API returns a icon-sized PNG file for the app identified by `{{uri}}`. This API returns a placeholder icon even if none were defined and even if the app doesn't exist.

## Get Spec (JSON)

`GET /api/namespaces/apps-svc/tree/spec?op=wait&input.uri={{uri}}&input.tag={{tag}}&field=output&raw-output=true&ctype=application/json`

This API returns a JSON representation of the swagger specification uploaded to an app. Both `{{uri}}` and `{{tag}}` are required, and if no match can be found the response will be an empty 200 OK. 

## Get Swagger (YAML)

`GET /api/namespaces/apps-svc/tree/get-swagger?op=wait&input.uri={{uri}}&input.tag={{tag}}&field=output&raw-output=true&ctype=application/yaml`

This API returns the original YAML representation of the swagger specification uploaded to an app. Both `{{uri}}` and `{{tag}}` are required, and if no match can be found the response will be an empty 200 OK. 

## Get Docs (Markdown)

`GET /api/namespaces/apps-svc/tree/md?op=wait&input.uri={{uri}}&input.tag={{tag}}&field=output&raw-output=true&ctype=text/markdown`

This API returns generated Markdown documentation for the swagger specification uploaded to an app. Both `{{uri}}` and `{{tag}}` are required, and if no match can be found the response will be an empty 200 OK. 

## Get Tags

`GET /api/namespaces/apps-svc/tree/tags?op=wait&input.uri={{uri}}&field=output&raw-output=true&ctype=application/yaml`

This API can return a list of all known tags for a given app `{{uri}}`. If no match can be found the response will be an empty 200 OK.

Sample Output:

```json
{
	"tags": [
		{
			"created_at": "2022-05-10T02:15:06.478118Z",
			"tag": "latest"
		}
	],
	"uri": "charlie"
}
```

## Get Short List

`GET /api/namespaces/apps-svc/tree/list?op=wait&field=output&raw-output=true&ctype=application/json`

This API returns a list of all known apps and all of their known tags, with a short description of what each does. It requires no arguments because it returns all results. If you receive abnormal data from this API something went wrong during the `rebuild` step.

Sample Output:

```json
{
	"bravo": [
		{
			"created_at": "2022-05-10T04:34:09.166283Z",
			"description": "Usql client for Direktiv",
			"tag": "latest"
		}
	],
	"charlie": [
		{
			"created_at": "2022-05-10T04:33:56.354059Z",
			"description": "Usql client for Direktiv",
			"tag": "latest"
		}
	]
}
```

## Get Long List

`GET /api/namespaces/apps-svc/tree/list-expanded?op=wait&field=output&raw-output=true&ctype=application/json`

This API returns an expanded list of all known apps and all of their known tags, with a short description of what each does. It requires no arguments because it returns all results. If you receive abnormal data from this API something went wrong during the `rebuild` step.
