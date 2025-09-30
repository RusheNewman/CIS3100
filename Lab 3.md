### **1. Creating Network Namespaces**

-  First, we use the ip netns add command to create two new network namespaces named netns1 and netns2. This creates two isolated network environments that you are then able to attach interfaces, routes, and so on to, that are separate from the host. The command itself binds a handle (basically a reference) to a new network namespace. This creates a namespace file in /var/run/netns/ so that you can reference that namespace later.

   ![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930115425.png)

- Next, we use a command to verify that we've indeed created these namespaces. This command probably just inspects the /var/run/netns/ directory and displays the names of the namespaces.

  ![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930120010.png)

### **2. Creating Virtual Ethernet (veth) Pairs**

- Now, we set up two virtual Ehternet pairs. This is essentially just a tunnel- what goes into one side comes out the other. So, if you send data to veth1_0, it immediately shows up in veth1_1. This enables bridging- one of these can be on the namespace side, and another can be on the host side, enabling communication between the two

  ![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930120402.png)

- Now, we're assigning the veth1_1 and veth2_1 interfaces to the netns1 namespace. This is exactly how we create a bridge- move one side of this "cable" to the namespace side while the other remains on the host side.

  ![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930120728.png)

## **3. Configuring Interfaces within Namespaces**

- This one activates the loopback interface lo inside the netns1 namespace. Inside the netspace, this interface is initially down, so we need to enable it so that localhost and internal communications are all working correctly. The "sudo ip netns exec netns*" part of the command just means that we want to execute what follows inside the network namespace.

  ![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930120953.png)

- Now, let's configure the IP addresses of the virtual interfaces. First, we activate (bring up) the interface veth1_0 on the host side
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930121357.png)

- Next, we assign it the "10.1.1.1" IP address on the host side. This makes it so that the host has an actual IP that it can communicate or route traffic to.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930121538.png)

- And we also bring up the interface on the namespace side
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930121613.png)

- Finally, we also assign this side an IP address (in this case, "10.1.1.2"). This means that packets sent to 10.1.1.2 should arrive inside the namespace's interface.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930121741.png)

- Then, we repeat the same steps again for the other interface. There's no point in repeating the explanations since it's all the same just with different names.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930121850.png)

- Next, we need to add the routes to the host's routing table because the host needs to know how to reach 10.1.1.2 and 10.1.1.3.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122121.png)

- Now, let's examine what we've done. Running the ip addr show command inside the netns1 namespace allows us to see all of the network interfaces inside the namespaces and their corresponding IP addresses. This way we can verify that, indeed, is present and UP (meaning it's activated)
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122121.png)

- And with the route show command, we can show the routing table. As we can see, everything is wired up properly.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122432.png)

- Everything looks similarly correct inside of the second namespace.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122526.png)

## **4. Testing Network Isolation**

- Now let's test that everything works. First, we try to ping the first interface from within it (its own interface). This works since, well, it's its own interface.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122712.png)

- However, we are't able to ping the other namespace interface from netns1. This doesn't work since the two namespaces are isolated from one another and aren't bridged.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122823.png)

- But if we ping the second namespace's interface from within the second namespace it works, just like when we pinged the first interface from inside the first namespace.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930122926.png)

- Pinging both of the namespace interfaces from the host, however, works, since we have a route pointing as veth1_0 and veth2_0.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930123034.png)

- Finally, let's clean up. First, we delete the interfaces by running the ip link del command, which removes the virtual networking device. Since veth1_0 is connected to veth1_1 the other one is also removed.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930123219.png)

- And then we detele the network namespaces themselves, which removes their reference from /var/run/netns/ and frees the namespaces, if no processes are using them.
  
![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Pasted%20image%2020250930123419.png)

## **Diagram**

![](https://github.com/RusheNewman/CIS3100/refs/heads/main/Lab%203%20Pictures/Untitled%20drawing(1).png)
