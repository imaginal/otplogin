# One Time Password Login as 2FA for SSH

Compatible with Google Authenticator (Android and iPhone)


## The idea

Even if SSH shas been compromised (by 0day exploit), we may require a second
factor to login.


## Why not use PAM?

PAM works inside the SSH process. If ssh authentication will bypassed through
an exploit (like CVE-2018-10993 libSSH authentication bypass) PAM will not help.
Therefore we will need a second, external authentication method.


## Requirements

- python
- PyOTP library
- PyQRCode (only for otpcreate)


## Installation

    # pip install pyotp pyqrcode
    # cp otplogin /usr/local/bin/


## Usage

1. Use `otpcreate` or `google-authenticator` from libpam-google-authenticator
  to generate secret key in ~/.google_authenticator
2. Put `command="otplogin"` before public key in .ssh/authorized_keys
3. Try ssh login
4. Profit!

WARNING: Do not close active SSH session until you are SURE that the 2FA
works as needed, otherwise you can completely block your ssh access.

CAUTION: `otplogin` starts after ssh authorisation process and it does not protect
ssh forwarding features (port-forwarding,x11-forwarding) so consider to disabling
them (in `/etc/ssh/sshd_config` or `~/.ssh/authorized_keys`).

For more protection against 0day attacks restrict ssh to execute `otplogin` only
(for example using AppArmor).


## Examples

	$ otplogin --help
	usage: otplogin [keyfile [command]]
	default keyfile ~/.google_authenticator


.ssh/authorized_keys

	command="otplogin - $SSH_ORIGINAL_COMMAND" ssh-rsa AAAB...

	no-port-forwarding,no-x11-forwarding,command="otplogin" ssh-rsa AA...


/etc/ssh/sshd_config

	Match user *
		DisableForwarding yes
		ForceCommand /usr/local/bin/otplogin /etc/otpkeys/$USER

	Match user foo
		DisableForwarding yes
		ForceCommand /usr/local/bin/otplogin - $SSH_ORIGINAL_COMMAND

	Match group admins
		AllowAgentForwarding yes
		AllowTcpForwarding no
		X11Forwarding no
		ForceCommand /usr/local/bin/otplogin ~/.otpkey

	# for scp
	AcceptEnv OTP_CODE LANG LC_*


## Security

1. Keyfile must be owned by user
2. Keyfile must not be accessible by others
3. Keyfile directory must not be writable by others
4. Shared secret minimum 80 bit (160 bit recomended)
5. User shell must be allowed in /etc/shells


## Bugs

1. `scp` may not working because scp ignores any user input.

	You can pass code via env OTP_CODE using -o SetEnv option,
	add `AcceptEnv OTP_CODE` in sshd_config on server side.

	Example:

		scp -o "SetEnv OTP_CODE=123456" srcfile user@host:


2. Debian/Ubuntu in case of error

		otplogin: /usr/bin/python: bad interpreter: No such file or directory

	just update first line of file to `/usr/bin/python3`


## Copyright

(c) 2020 Volodymyr Flonts <flyonts@gmail.com>


## License

MIT
