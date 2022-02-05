## Pinguino Automated Deployment

I maintain a Discord bot called Pinguino that, at the time of writing, has had 20 releases. The bot is hosted on the my DigitalOcean droplet.

I used to deploy new versions of Pinguino to my droplet manually - `ssh`, `wget` the file URL, replace the old file, reboot the server to automatically run the bot.

This would not do for me! It's too slow and painful to do it manually everytime I create a new release, so I decided to automate it.

### The Automation

A simple overview of the process:

1. Create a new release on GitHub
2. Push the newly built JAR to [the builds repository](https://github.com/JamCoreDiscord/builds)
3. Let GitHub Actions do the rest!

But what does actions actually do? Let's have a look at `publish.yml` within the builds repository:

```yml
name: Pinguino Publish
on: push

jobs:
  publish:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2
      - name: Get Added Files
        id: files
        uses: jitterbit/get-changed-files@v1
      - name: Add Files to ENV Variable
        run: |
          for changed_file in ${{ steps.files.outputs.added }}; do
            echo "latest_build_url=https://github.com/JamCoreDiscord/builds/tree/main/${changed_file}" >> $GITHUB_ENV
          done
      - name: Make API Request
        run: |
          curl -X POST -H "Content-Type: application/json" -d '{"build_url": "${{  env.latest_build_url  }}"}' ${{  secrets.API_UPLOAD  }}
```

This is what's going on in this file:

- `name: Pinguino Publish` - This is the name of the job.
- `on: push` - This is the event that triggers the job, in this case we run this file every time a new push is creatd.
- `runs-on: ubuntu-20.04` - This is the OS that the job will run on.
- `name: Get Added Files` - This is the step that gets the files that were modified in the push, via the action `jitterbit/get-changed-files`.
- `name: Add Files to ENV Variable` - This is the step that adds the file to the environment variable `GITHUB_ENV`. The step doesn't look for a specific file, it simply goes through the added files and adds the latest one - primitive but it works.
- `name: Make API Request` - This is the step that makes the API request to the Pinguino Deployment API, using the `curl` command and GitHub repository secrets to hide the URL.

The Pinguino Deployment API is an _extremely_ simple API running on my droplet. The source can be found [here](https://github.com/JamCoreDiscord/deployment). The API uses NodeJS and Express to provie a single endpoint (excluding the `/ping` endpoint), `/notify`.

This is what the code looks like for that endpoint:

```js
//Input will be: {"build_url": https://github.com/JamCoreDiscord/builds/tree/main/...}
var url = req.body.build_url;
url = url.replace("tree", "raw");
console.log("Recieived update with URL: " + url);

res.sendStatus(200);

if (url.includes("Pinguino") && url.includes(".jar")) {
    childProcess.exec(
        "sh updatePinguino.sh " + url,
        (error, stdout, stderr) => {
            console.log(stdout);
            console.log(stderr);
            if (error !== null) {
                console.log(`Error Updating Pinguino: ${error}`);
            }
        }
    );
}
```

This code simply checks the URL to see if it contains the word `Pinguino` and the file extension `.jar`. If it does, it runs the `updatePinguino.sh` script, which will download the file, replace the old one, and restart the droplet. This is that script:

```bash
#!/bin/sh

cd ~/Pinguino
rm Pinguino.jar

wget $1

for x in *.jar; do
    mv "$x" "Pinguino.jar"
done

sudo reboot
```

So that's how Pinguino get's automatically updated when I make a new release. I eventually want to phase out the builds repository and just use a GitHub webhook to ping the API, but I don't really have time to work on that right now. I'll make a new post once I do get that working :). 

Bye for now - Jamalam.