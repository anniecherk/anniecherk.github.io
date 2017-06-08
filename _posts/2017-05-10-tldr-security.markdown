---
layout: post
title: "tldr; Security"
date: 2017-05-10
categories:
---
## See something? Say something!

In light of recent security headlines- notably [the global ransomware virus](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack) and [the fake google doc phishing attack](https://www.theverge.com/2017/5/3/15534768/google-docs-phishing-attack-share-this-document-with-you-spam), both of which happened this week, and [the large scale DDoS](https://www.incapsula.com/blog/malware-analysis-mirai-ddos-botnet.html) (distributed denial of service) attack earlier this year- I've heard a lot of security advice that I completely disagree with, so here are my two cents on the best/easiest way to keep your machines safe:

**1) Keep your machine updated!** This means that when your computer says "should I install updates?" you say YES! If you do nothing else, do this.

**2) Use a password manager.** I use [1password](https://1password.com/), but if you want a free password manager I would try [lastpass](https://www.lastpass.com/). Password managers are a *great* idea- more on this in a minute! It only takes an hourish to set up, and after that it takes no extra time or effort and provide *a lot* of security- totally worth it!

Those are the big ones. I'll talk about other useful/easy security things you can do a little lower on the page, but if that's all you wanted to read, please just keep your machine up-to-date and set up a password manager!

# Why?

### Keeping a patched machine

So here's the thing about the [WANNACRY ransomware attack](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack) this weekend- it was a vulnerability that Microsoft knew about and had patched weeks ago. The computers that were affected (reportedly 75,000 of them!!) were not updated or running a very old operating system. The difference between being vulnerable in a large attack and being immune could be updating your machine, and it's really easy to do, so please please do it.

For the same reason it's important to be using an actively maintained operating system. Sometimes it's not possible to update your operating system because of software dependencies or old hardware, but if you can't update your operating system you're putting yourself at risk. For instance, Microsoft released [155 security updates last year](https://technet.microsoft.com/en-us/library/security/mt637763.aspx); so unless you have a *really* good reason, you should be using an officially maintained operating system.

Here's how that attack worked: there was a bug in a Windows internet protocol, and this bug allowed attackers to 1) get access to the machine through the internet, and 2) get administrator privileges on the machine. This allows the attackers to encrypt all the files on the hard drive and ransom them.

All programs have bugs; many large important programs like operating systems have important, exploitable security bugs. That's just the way it is. Security is really hard to get air-tight correct, and to err is human. There are many people who are actively looking for these bugs, and many companies will pay rewards for telling them about serious bugs first so that they can fix them.

When a security vulnerability is made public, like the one in this ransomware attack was a few weeks ago, the affected software companies publish a patch that fixes the bug as soon as possible (often as the bug is being publicly disclosed). You can't protect against bugs that haven't been disclosed, but it's reckless to not protect against bugs that have been patched; that's why it's really important to keep your machine updated. Meaning that if you haven't updated your machine in a year, all the relevant security bugs that have been found in the last year could be used against you.


### Passwords

**Q: Do I need to use a different password for every website?**
**A: Yes**, and using a **password manager** makes this really easy. I use [1password](https://1password.com/) to generate a different random password for every website I use.

You need to use unique passwords because sites that store your password get attacked, and [your password might get leaked](http://fortune.com/2016/05/18/linkedin-breach-passwords-most-common/). Using unique passwords minimizes damage by keeping your other accounts secure. If you use a different password for each site there's no reason to change all your passwords just because one gets hacked. If, on the other hand, you use the same password for every site and any of them get hacked, then the attacker can try logging into all your other accounts with that password. That's why it's really important to use unique passwords, and again, if you use a password manager having different passwords is effortless.

**Q: What makes a secure password?**
**A: Length is the most important part of a secure password.** 4+ unrelated words or a long string of random characters will be more effective than replacing letters with symbols or numbers or changing your short password often.

Here's how an attacker hacks your password: they guess-and-check your password as many times as they want in an automated way. They'll try every word in the dictionary, and variations of every word, to see if that was your password.

Here's the problem with replacing characters in your password with numbers of special characters (like changing `"pancake"` into `"p@ncak3"`): password hackers also know these rules. They can check various spellings of words in their dictionaries. Using these replacements makes their job a little bit harder, but not much.

So here's a better strategy: [pick 4+ unrelated words](https://xkcd.com/936/). Again, password managers can do this for you. I just popped mine open and it suggested "folio lapel timely earwig lentil". This is a better strategy because each extra word you use makes the search exponentially more difficult (since there are way more english words than common character replacement options), so this is a *much* better defense than random character replacement.

Another good strategy is using a long string of random characters. Again that becomes exponentially harder to hack with every additional character, and a password manager can automagically generate & manage this for you.

**Q: Okay, but won't a long password be annoying to type in?**
A: Password managers **automatically fill in** your passwords!

**Q: What if I need to login on my phone?**
A: Depends on the password manager, but **1password is really easy to use from my phone.**

**Q: But what if something catastrophic happens to my password manager? Will I be locked out of everything?**
**A: Know your gmail password for redundancy.** I have a lot of faith in my password manager, but I'm also prepared in case something goes terribly wrong and I can't access my passwords. My solution is I remember my gmail password; I can always reset all my other accounts' passwords from my gmail. (This is exactly the reason it's especially important to have a strong gmail password!)

**Don't keep a passwords.txt file.** If you've got a plaintext file on your computer where you write down all your passwords, that's basically exactly what a password manager does BUT a password manger will also encrypt your passwords so they're safe against attackers, generate strong passwords for you, and automatically fill them into webforms!

In conclusion, password managers are great and *so* worth the effort of setting them up! Invest in a password manager today!

# More security advice

**3) Phishing attacks are very effective, and hard to protect against.** A phishing attack often looks like someone sending you an email that has a malicious attachment. The problem is that these attacks are often sophisticated- they may look like an email from someone in your contacts list, or from a social media site- but actually they're from an attacker trying to get you to click a link or download a virus. In my experience gmail is pretty good, but not perfect, at filtering these from your inbox; be very suspicious of any email that seems surprising or that you weren't expecting.

One thing that helps protect against phishing attacks is always showing file extensions. Here's how to set that up [on windows](https://support.microsoft.com/en-us/help/865219/how-to-show-or-hide-file-name-extensions-in-windows-explorer) and [on mac](https://support.apple.com/kb/PH25381?locale=en_US). This will let you know whether that `puppy.jpg` file is secretly a `puppy.jpg.exe` executable virus.

**4) Backup your machine.** This is pretty easy, [especially on a mac](https://support.apple.com/mac-backup). The benefit of backing up your machine is that if you do get a ransomware virus that encrypts all your files, you can say fuck you, reinstall your OS and restore from backup. They can't ransom your files because you've got your files. This also means you won't lose all your files if someone walks away with your laptop bag at a restaurant.

**5) Turn on hard disk encryption.** If your hard disk is encrypted then if someone steals your laptop, your data is all garbled and therefore safe. I'm including it mostly because it is *so* [easy to do on a Mac](https://support.apple.com/en-us/HT204837) that there's no reason not to do it. You can also encrypt [Android phones](https://www.howtogeek.com/141953/how-to-encrypt-your-android-phone-and-why-you-might-want-to/) and encryption is on by default for newer iPhones (and [you can turn it on](https://ssd.eff.org/en/module/how-encrypt-your-iphone) on older iPhones).

**6) Use 2 factor authentication.** 2-Factor Authentication means that to log into an account you need to "know something" (your password) and also "have something" (usually your phone). If someone tries to log into your account from some weird non-trusted computer, the site will send a message to your phone asking if that was really you- and let the person into your account only if you say "yup, that's me!" You can set up 2-factor for lots of popular sites like gmail, amazon, dropbox, facebook and  twitter. It's easy to set up and will go a long way to protecting you against hackers even if they manage to get your password.

**6) Use Privacy Badger & an adblocker.** [Privacy Badger](https://www.eff.org/privacybadger) is a great chrome extension from EFF and it's a one-stop-shop for lots of great security / privacy settings, and I highly recommend it, especially because it's so easy to set up. Ads can sometimes be a security problem because they will sometimes run arbitrary javascript which can be malicious.

**7) Use HTTPS Everywhere.** HTTPS means you've got a secure connection. There's a really awesome Chrome extension called [HTTPS Everywhere](https://www.eff.org/https-everywhere) also from EFF that will make sure you always get the secure version of a website if one exists. This is really really especially important if you're sending credit card information or downloading anything!

One caveat: Sometimes one of my chrome extensions will "break" a website, like sometimes webforms don't work correctly. When this happens I just open the page in incognito mode (which disables all extensions) and continue about my day, no sweat.



### HTTPS & Downloading anything

HTTPS protects you from "man in the middle" attacks. When you connect to a website, you and the website send "packets" of data back and forth which your browser uses to see what the website should look like. You can see all the packets that are flying around on your local network (from everyone connect to the same internet connection as you), so if you're interacting with a webpage without encryption I can see all the packets that you are sending and receiving.

This has two problems- 1) I can see your data, and do things like steal your credit card information when you enter it into a website, and 2) if I'm between you and the website I can intercept packets that the website sends to you and modify or replace them. This leads to really nasty attacks where you might think you're putting your password into a website but you're actually just handing your password to the man in the middle.

HTTPS protects you from both of these problems by encrypting your communication between you and the server. This means that if anyone tries to read your messages it just looks like a meaningless jumble, and no one can send you a fake webpage because it's impossible for them to encrypt it in a way that looks the same as the real website.

[HTTPS Everywhere](https://www.eff.org/https-everywhere) is a really awesome browser extension that will make sure you're getting the safe version of webpages you visit and can warn you if you're on an insecure webpage. It's great and I think you should get it.

HTTPs also makes sure you're getting the download that you asked for, instead of some other file that a man-in-the-middle is handing you. This doesn't protect you from downloading something that's a virus; in general I try to be very cautious and selective about what I download. Don't download anything, especially not executables, from websites you don't trust, and be careful about links that you open.

**9) Your email & phone are the keys to the kingdom.** One last take-away is that you probably use your email and/or your phone to authenticate all your other accounts. If one of those is compromised, the game is up. This means it is extra important to have a strong secure password on your email, and a screen lock or password on your phone. Using 2-factor authentication on your email will protect it against being hacked even if someone has your email password.


# In conclusion

**Patch your machine & use a password manager!**

Good night & good luck!
