# EC2 Minecraft Server

## Installation
1. Create an AWS account with a payment method (this costs a small amount of money to use)
1. Create a local SSH key pair (`ssh-keygen`, use default settings on windows)
1. Import created SSH public key (https://eu-west-2.console.aws.amazon.com/ec2/v2/home?region=eu-west-2#ImportKeyPair:)
Local public key will be `C:\Users\yourname\.ssh\id_rsa.pub`, 
1. Create an account with duckdns (https://www.duckdns.org/domains)
1. Register a domain name on duckdns
1. Create CLI access key (https://console.aws.amazon.com/iam/home?region=eu-west-2#/security_credentials) Make sure to make a note of the displayed secret. 
1. Install AWS CLI (https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html)
1. Configure AWS command line interface with CLI access and secret key from above: (https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-windows.html), use region eu-west-2
1. Download cloudformation.yaml from this repo
1. In the same directory deploy the server:

``	
	aws cloudformation deploy `
	--stack-name minecraft-server `
	--template cloudformation.yaml `
	--capabilities CAPABILITY_NAMED_IAM `
	--parameter-overrides `
		SshKeyName=<MYKEYNAME> `
		DuckdnsDomain=<DUCKDNSHOST> `
		DuckdnsToken=<DUCKDNSTOKEN> `
		MinecraftUsername=<MINECRAFTUSERNAME> `
		RconPassword=<MAKEAPASSWORDUP> 
``

1. It will take approximately 5 minutes for the server to be ready to use the first time.
1. Once the deployment has completed you will not need these tools again, the instance can be started and stopped from the aws console (https://aws.amazon.com/console/)

## Backups, startup and shutdown
The server can be started and stopped with the aws console (https://eu-west-2.console.aws.amazon.com/ec2/v2/home?region=eu-west-2#Instances:)
The server will automatically shut itself down after a configured number of seconds of with no-one logged into minecraft, default is 1 hour, can be configured with MinecraftIdleShutdownSeconds=XXXXX on deployment
The server will always take a backup in S3 (https://s3.console.aws.amazon.com/s3/home?region=eu-west-2), multiple versions will be retained for one week, with the most recent version being retained indefinately.

## Costs
Small, at the time of writing a t3.medium instances costs around half a dollar cent per hour of use.
https://aws.amazon.com/ec2/pricing/on-demand/ 
There will be some additional costs associated with backup storage, but they should be very small or nil.
t3.medium may not be enough it has 2 cores and 4GB of RAM, more cores might be needed but the cost jumps dramatically, (t3.xlarge has 4 cores and 16GB RAM and costs 16 dollar cents an hour)

## Minecraft configuration
Minecraft server files can be found in /home/minecraft/game-data, a script /home/minecraft/server-command.sh is provided to issue standard minecraft server / admin commands from an ssh session. (https://www.ign.com/wikis/minecraft/Admin_and_Server_Commands) The default configuration is thus
* 3GB ram allocated to java for the server
* 5 users maximum
* users must be whitelisted
* difficulty normal
* gamemode survival
* The minecraft user provided in the deployment above is added as an operator and can add additional users using standard minecraft commands (`/whitelist add AMINECRAFTUSERNAME`), commands can be used from within the game or from the server-commands.sh script.

## SSH access
When running access the server with:

	ssh ec2-user@<DUCKDNSHOST>.duckdns.org

Becuase the IP address of the server will be different each time its accessed this local configuration will be needed (in `C:\Users\yourusername\.ssh\config`):

	Host <DUCKDNSHOST>.duckdns.org
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null    

## Notes
To destroy and recreate entire stack (will need to empty the S3 bucket manually first):

	aws cloudformation delete-stack --stack-name minecraft-server-trunk;aws cloudformation wait stack-delete-complete --stack-name minecraft-server-trunk;.\deploy-trunk.ps1

