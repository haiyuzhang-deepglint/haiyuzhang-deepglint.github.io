Google Chrome 浏览器不能保存密码的问题:

Link: http://plasmasturm.org/log/chromepwstore/

his is a kind of post that people used to write back in the heady early days of blogging and a more communal web: putting something out there to help Google help other people.

## The problem

For some time I had been having an irritating persistent failure with Google Chrome that I could not find an answer for:

- After logging into some website, it would offer me to save the password, as usual.
- I would click on the save button.
- Chrome would not show any kind of error.
- But the password would not be saved:
- It was not filled in automatically next time I went to the same site.
- No password at all showed up in the list on `chrome://settings/passwords` – the list just stayed blank no matter what I did.

Mysteriously, a handful of passwords did get stored, somehow, somewhere. Chrome could fill those in, even as it woudn’t list them on the settings screen. I checked the MacOS keychain and did not find them there, so they had to be stored by Chrome, even though it refused to show them to me.

## The quest

Searching the web about my problems, almost all answers I could find related to the case of people who are logged into Google within Chrome and use its password syncing service… which I don’t. I simply want my passwords saved locally.

The few answers I did find that seemed to relate to my situation invariably suggested resetting one’s profile. Now, that approach does appear not to be mere superstition: of the people I found who had this problem, the ones who reported back all wrote that resetting their profile fixed the problem. So I had a way of making the problem go away – but I also have a lot of data in my profiles. It’s not just my bookmarks. I have tweaked many of the settings, individually for each profile (the whole point of using profiles, after all), and I also use a number of extensions, many of which themselves have extensive configurations. Recreating that all is a big task.

**I want password auto-fill fixed while keeping my profiles intact. I am only willing to lose my stored passwords.** (Because I save them in a password manager first anyway.)

So I went poking around in the directories where Chrome stores its profiles and other user data. There’s no need to look far: there’s a file called `Login Data`. This is an SQLite database (like most of Chrome’s user data files). It can be opened using the `sqlite3` command line utility and examined using SQL queries. I did that, and there they were, my mysteriously saved passwords… plus a bunch more.

I also discovered that some of the data in those tables is scrambled in some form. Presumably there are several separatedly stored pieces of data required to unscramble that data, and some of those pieces somehow become mismatched on my system – I don’t know how nor why, and was too lazy to research. All I cared was that this looked like the right vicinity.

As an experiment, I moved the `Login Data` file and its `Login Data-journal` pair out of one profile… and bingo, password auto-fill started working there as expected. After saving a password, Chrome would subsequently successfully auto-fill it as well as list it on the saved passwords screen.

Good enough for me.

## The solution

Deleting the files `Login Data` and `Login Data-journal` from a profile fixes password saving in that profile – without affecting any other data in it. A full profile reset is not necessary – you can reset just the password storage by deleting just the files that it uses.

This does mean you lose any passwords you had stored previously, unfortunately. But since you cannot really access them any more anyway, that data loss has effectively already happened by the time you delete the files.

## Instructions

1. Quit Chrome.

2. Go to the directory where Chrome stores its user-specific data, below your user home directory:

   - Mac

     `~/Library/Application Support/Google/Chrome`

   - Linux

     `~/.config/google-chrome`

   - Windows

     `%UserProfile%\AppData\Local\Google\Chrome\User Data`

3. From there, go into the directory called `Default` if you want to fix your main profile, or into `Profile 1` or `Profile 2` etc. to fix one of your extra profiles.

4. Delete the files `Login Data` and `Login Data-journal`.

5. Repeat for other profiles as necessary.