Thinking about a title for this post, I was between this and *How to delete 6 servers in one move*. I decided to keep the first one, because it makes it clear that I am going to talk about bash scripting and that something went reaaaally bad!

The case that created the need for the script, was for a SaltStack infrastructure, that contained a master node and about 10 minion nodes. The minions - nodes that the master can send commands to - had to expose some services over TLS, so a Certificate Authority had to be created and used to sign the minion CSRs. With about 4 TLS services on every node, we had about 40 CSRs that had to be transfered to the CA, get signed and then sent back to the right minion.

This is the point where I stop talking about the specific case and start discussing the main point of this post: rough and dangerous as it can be, the command line provides the way to execute complex tasks, by combining various propriate tools, and automate procedures, that when manually executed would be tremendously long and error prone. Manually collecting 40 files from various nodes, performing an action on them and sending them back is a great example of such a procedure.

So I developed a 150-line, monolithic script, that was divided in logical sections only by comments, that described what the following lines were about to do. No modularity was inherently provided - you could run the whole thing or nothing, which was the biggest problem with the specific script. I also adequately tested the procedure - while I was developing the script - and all seemed to run smoothly.

Until I ran the the script part by part. Of course there was no such thing as a modular part, so I just commented out the logical sections, starting with a commented line. But since the script was not designed to run like this, dependencies existed between parts of code I tried to run independently. This, inevitably lead to the following:

```
...

## Create temporal certificate directory in minions
#minion_home="/home/ubuntu"
#minion_cert_dir="certs"
#for minion in ${minions[@]}; do
#    salt $minion cmd.shell "mkdir -p ${minion_home}/${minion_cert_dir}"
#done

...

# Remove temporal certificate directory in minions
for minion in ${minions[@]}; do
    salt $minion cmd.shell "rm -rf ${minion_home}/${minion_cert_dir}"
done
```

![Coding Horror](../img/coding-horror.png)

Developing the script in a monolithic way and then running it modularly was clearly a big mistake, that lead to unitialized variables, that when used in an equivalently wrong command, executed

    rm -rf /
    
on 6 of the infrastructure minions. There was no way to save them, so they had to be set from the beginning. As Jeff Atwood wrote:

> You're an amateur developer until you realize that everything you write sucks. YOU are the Coding Horror.

After this I was thinking of abandoning the catastrophic script, but it was no better choice to manually perform a highly error prone procedure either. What I did instead, was to analyze my code flaws and refactor it to a more secure and elegant shape. These actions were:

Since the biggest problem with the script was that it could not run in a modular way, I introduced functions to divide the file to independent modular sections. I tested each function separetely and made sure that all the variables and parameters used were properly initialized. A main function at the end of the file included all the script logic, calling the available functions with the proper sequence. There was no other command in the script, but a single `main`, in order to avoid *forgotten* commands between a vast amount of functions.

The disasterous command ran because I tried to delete a temporal directory, so I removed all the `rm` commands from the script and provided a *delete_temporal_directories* function, that would check if the directory variables have the correct values and only then remove the corresponding directories.

Something similar could happen with `mv` too, so I reshaped

    mv ${certs_dir}/* /etc/ssl/mycerts
    
to

    mv ${certs_dir}/*.cert.pem /etc/ssl/mycerts

In order to provide clear, readable code I used descriptive function names, that showed the purpose of the function. I removed all comments, except the ones that supplementary added extra information about the function.

```
# Server pems include private key + certificate
function create_server_pems_in_minions {
```

Of course the biggest lesson I learned is to never forget all good practices, even when developing under pressure to meet a deadline. After all, if something catastrophic happens, the time you will spend to recover will be much more, than the time saved to finish a script earlier.

I felt completely disapointed and emberased by the trouble I created, and this is why I tried to find the root cause of the problems. Now, I feel really good that I work with people that do not discourage me to occasionally make some mistakes. :)


