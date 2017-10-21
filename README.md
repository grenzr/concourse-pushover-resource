# Concourse pushover resource

[Concourse CI](https://concourse.ci) Resource to send notifications to the [pushover app](https://pushover.net/)

The resource is based on [smuggler concourse resource](https://github.com/redfactorlabs/concourse-smuggler-resource) to quickly implement it with a small script.

## Resource Type Configuration

```
resource_types:
- name: pushover-notification
  type: docker-image
  source:
    repository: redfactorlabs/pushover-notification
    tag: latest
```

## Source & params Configuration

The following parameters can be passed in the `source` or in `params` of `put`

 * `app_token`: Required. Token of app configured in https://pushover.net/apps/build
 * `user_key`: Required. the user/group key (not e-mail address) of your user (or you), viewable when logged into our [dashboard](https://pushover.net/dashboard)
 * `message`: Optional. The message to send to the user. Can use environment variables from BASH. There are some variables about the current build, as used in the example below.
 * `atc_external_url`: Optional. Used in the default message.

You can add any additional parameter and use it in the message with `$SMUGGLER_<param_name>`

## Example:

```
resource_types:
- name: pushover-notification
  type: docker-image
  source:
    repository: keymon/concourse-pushover-resource
    tag: latest

resources:
- name: pushover-build-fail-notification
  type: pushover-notification
  source:
    atc_external_url: https://concourse.mycompany.com
    app_token: {{pushover_build_notifications_app_token}}
    user_key: {{pushover_group_key}}

jobs:
- name: some_important_job
  plan:
  - task: fail
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: alpine
      run:
        path: sh
        args:
        - -ec
        - |
          echo 'Something bad happened :('
          false
    on_failure:
      put: pushover-build-fail-notification
      params:
        message: |
          OMG!!! the important job failed!!! ARGHHH!

          Check it out ASAP here:

          ${SMUGGLER_atc_external_url}/teams/${BUILD_TEAM_NAME}/pipelines/${BUILD_PIPELINE_NAME}/builds/${BUILD_NAME}
```

## Important security limitation!

> IMPORTANT: the current implementation does an shell `eval` to expand variables.
> That is prone to [Command injection](https://www.owasp.org/index.php/Command_Injection).
>
> This is not too much risk when running concourse, as the concourse users are trust, but
> be aware of this.  It is easy to fork this repo and change the file `smuggler.yml` to
> remove that eval.

## Contributing

It is easy to extend this resource, by editing the file `smuggler.yml`.

The API references for the messages is here: https://pushover.net/api
