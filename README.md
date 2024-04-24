# PM2 Deployment

NodeJS/Typescript deployment with [PM2]("https://github.com/Unitech/pm2"). This action has support for SSH keys for those who would rather do that than passwords.

## What I changed:

1. I updated the versions of the appleboy/ssh-action actions used as the latest versions have support for SSH keys but many of the existing PM2 deployment actions I found were not yet updated.

2. I found that the popularly used appleboy/scp-action did not work well for my hosting and permission setup, so I opted to use rsync instead.

3. I plan to re-use this same action for managing staging and production environments so I added additional inputs and conditional logic to, for example, copy files I've already staged and reviewed over to my production directory.

## How to use the action?

### Standard Deployment

```yml
steps:
  - name: Deploy app
    uses: keyknee/pm2-deployment-key@main
    with:
      remote-target-path: "/deployment/api"
      host: 12.34.56.78
      username: ${{ secrets.prod-user }}
      port: 2080
      key: ${{ secrets.prod-key }}
      pm2-id: "api"
```

### Copying from remote-source to remote-target

```yml
steps:
  - name: Deploy production from staging
    uses: keyknee/pm2-deployment-key@main
    with:
      copy: "true"
      remote-source-path: "/staging/deployment/api"
      remote-target-path: "/production/deployment/api"
      host: 12.34.56.78
      username: ${{ secrets.prod-user }}
      port: 2080
      key: ${{ secrets.prod-key }}
      pm2-id: "api"
```

## How does it work?

The action does the following:

1. Copies the repository contents to the remote server on the specified folder
2. Runs `npm ci`
3. Runs `npm run build` (optional)
4. Runs `pm2 reload` the ecosystem.config.js
5. Runs `pm2 reset <id>` to set 0 the number of restarts

## Input arguments

- `remote-target-path` - Where do you want to copy the files to?
- `host` - \_What's the host IP address?
- `username` - What's the username that you're going to login into
- `port` - What's the port of SSH? (default: `22`)
- `password` - What's the password of the user?`` (Note: in the future SSH Keys will be supported)
- `key` - the named secret you created for store your private key.
- `build` - Set to "true" to build your typescript app
- `copy` - set to "true" to copy existing remote files to another remote location. This is helpful in instances where you may have one workflow that runs this action with `copy` omitted - or false - that builds and deploys to a staging location, and then another that runs with `copy` true to copy the staged files to the location used for production.
- `remote-source-path` - if `copy` is set true, you'll need to use this parameter to specify the source file location.
- `pm2-id` - What's the ID/Name of the PM2 application?

> Note: I recommend to use ecosystem.config.js in your project, you can envs, interpreters, node args, num of instances and more options. Here there are more content [about it](https://pm2.keymetrics.io/docs/usage/application-declaration).

> Good Practices: Why do I use reload instead restart? Restart is a hard way to stop and start a web server.
