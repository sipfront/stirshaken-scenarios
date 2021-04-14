# STIR/SHAKEN Test Suite

## WARNING

This version still has `sipp.opensipit.sipfront.org` as x5u URL scattered all around the code and
scenarios and will be made configurable in the future.

For now, run a quick hacky search/replace over it, like so:

```
perl -pi -e 's/sipp.opensipit.sipfront.org\/cert.pem/url.of.your\/cert.pem/' *
perl -pi -e 's/sipp.opensipit.sipfront.org\/cert.pem/url.of.your\/cert.pem/' helpers/*
perl -pi -e 's/sipp.opensipit.sipfront.org\/cert.pem/url.of.your\/cert.pem/' scenarios/*
```

## Prerequisites

On Debian buster, install the dependencies as follows.

```
# libks and libstirshaken1 are from the freeswitch repo
apt-get update
apt-get install -y gnupg2 wget lsb-release
wget -O - https://files.freeswitch.org/repo/deb/debian-release/fsstretch-archive-keyring.asc | apt-key add -

cat <<EOF > /etc/apt/sources.list.d/freeswitch.list
deb http://files.freeswitch.org/repo/deb/debian-release/ buster main
deb-src http://files.freeswitch.org/repo/deb/debian-release/ buster main
EOF

apt-get update
apt-get install -y libks libstirshaken1 libuuid1 libcrypt-jwt-perl libdata-uuid-perl libcryptx-perl
```

## Serving the certificates

Copy your certificates and private keys into `certs/sp`, so the test suite can find them and use them to
sign the messages.

Note that the file names in certs/sp/ must match the names below!

```
# the valid certificate
cp /path/to/cert.pm certs/sp/cert.pem

# again the valid certificate, but we'll serve this on
# a different port, and to avoid caching on the client
# side, we use a different name
cp /path/to/cert.pm certs/sp/copyofcert.pem

# the private key belonging to your valid certificate
cp /path/to/priv.key certs/sp/priv.pem

# the ca certificate for your valid certificate
cp /path/to/cacert.pem certs/ca/cacert.pem

# an expired certificate with the same key as above
cp /path/to/expired.pem certs/sp/expired.pem

# a certificate valid only in the future, with same key as above
cp /path/to/future.pem certs/sp/future.pem

# a certificate signed by an untrusted CA
cp /path/to/untrusted.pm certs/sp/untrusted.pem

# and the corresponding priv key for the untrusted cert
cp /path/to/untrusted.key certs/sp/untrusted.key

### here go some auto-generated fake certificates we'll need

# an empty certificate
> certs/sp/empty.pem

# a certificate full of garbage
> certs/sp/garbage.pem
for i in $(seq 0 4096); do echo -n a >> certs/sp/garbage.pem; done
```

The certificates referenced in the x5u of the PASSporT and the info param of the Identity
header must be provided via http also. There is a helper script `helpers/run-httpserver.sh`
using Python's built-in http server, which you can use to serve certs on both port 80 and 8080,
which are used throughout the tests:

```
# in one terminal
helpers/run-httpserver.sh 80

# in another terminal
helpers/run-httpserver.sh 8080
```

## Usage

You have to set up your environment per test target, then you can run each test automatically.
If one test hangs, just CTRL+C and investigate the error and message log for the test number in
`runs/$TARGET`.

```
# set your target here
TARGET="kamailio.opensipit.sipfront.org"

# set your caller id here
CALLER_ID="439991001"

# set your called id here
CALLED_ID="439991002"

mkdir -p "runs/$TARGET"

cat <<EOF > "runs/$TARGET/caller.csv"
SEQUENTIAL
$CALLER_ID;$TARGET;$CALLER_ID
EOF

cat <<EOF > "runs/$TARGET/callee.csv"
SEQUENTIAL
$CALLED_ID;$TARGET;$CALLED_ID
EOF

for test in *.sh; do ./$test "$TARGET"; sleep 1; done
```

## SIPp version

This testsuite currently uses a binary version of SIPp in helpers/sipp including
STIR/SHAKEN support. You have to install libstirshaken to be able to run it.

We'll provide a patch against upstream SIPp (and will try to get it merged into
upstream) as soon as possible.
