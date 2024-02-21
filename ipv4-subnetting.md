# IPv4 Subnetting

Hey there 👋🏼, fellow sudoers and other readers! Welcome to my first (and hopefully not last) sudorealm blog post.

In this article we will discuss a recent project of mine, the [Subnet Calculator](https://teobourloglou.github.io/subnet-calculator) . It was not long ago that I began my journey in networking and one of the first concepts I stumbled upon was subnetting. I wanted to build something relevant to this in order to understand better the logic behind it and secondly to sharpen further my Javascript (AlpineJS) skills (still a long way to go).

![Application Screenshot](https://sudorealm.com/post_assets/pZ9czczpoEDt5EYfShqlcjdjXJMeKf3sxRExWWYV.png)

But what is subnetting? If this sounds new, don't worry, we're in this together, let's take it one step at a time:
## Table of Contents

1. What is Subnetting anyway?
2. Dive a bit deeper
3. Let's build it

## What is Subnetting anyway?

Imagine a large city (🏙️) with houses (🏠) and neighborhoods. Now, in the world of networking, we have a similar concept. Think of the entire city as a network, and each house as a device, like a computer or a printer, connected to that network. 

Each device needs its own unique identifier called an **IP address** like each house in a city has a distinct address (🪧). It ensures that data sent or received by that device reaches the right destination within the network, just like mail being delivered to a specific house.

Then there is also the network IP address. What is this? Well we can think of the city's overall address as the "network address". This IP address uniquely identifies the entire network, much like a city's address pinpoints its location on a map (📍).

However, in a city, you don't just have houses; you also have neighborhoods (🏘️). These neighborhoods help organize the city into more manageable sections. In networking, a subnet is like a neighborhood. It's a way to group devices together based on their IP addresses. This grouping provides better control, security, and resource optimization, making the overall network more scalable and easier to maintain. *This is what subnetting is*, the organization of a city (network) into smaller neighborhoods (subnets).

Everything clear until now? Let's sum it up before continuing:
- Network = City 🏙️
- Subnet = Neighborhood 🏘️
- Device = House 🏠
- IP Address = House Address 🪧
- Network IP Address = City's Location on the map 📍

Pretty easy right?

![Destroyed City](https://media.giphy.com/media/l2Jeja7dECQ0W0Yec/giphy.gif)

## Dive a bit deeper

In order for us to be able to perform the subnetting we need two things:
- The Network IP Address
- The Subnet Mask

Let's talk about the subnet mask. In our city analogy, the subnet mask is like the boundary or the fence that separates one neighborhood from another. It defines which part of the city belongs to a particular neighborhood. The subnet mask is a series of 4 numbers in the range 0-255, divided by dots e.g. `255.255.255.0`

It's important to refer also to the CIDR notation of the subnet mask, which is the number of continuous '1' bits of the subnet mask represented in binary.

```
255.255.255.0 = 11111111.11111111.11111111.00000000
CIDR Notation: /24
```

For example, let's say the city's IP address is 192.168.0.10, and a neighborhood (subnet) has a subnet mask of 255.255.255.0. This means that all houses within this neighborhood will have IP addresses that start with 192.168.0.x, where 'x' can be any number from 1 to 255. So our subnet has the range 192.168.0.1 - 192.168.0.255

How can we know this? The calculation of the subnet range consists of two parts. Representing the IP address and the subnet mask in binary and performing two bitwise operations to find the first and last address of the network.

The first address of the network, also called **Network Address** or Network Prefix is calculated by performing the bitwise AND operation of the IP address and the subnet mask:

```
192.168.1.10 = 11000000 . 10101000 . 00000000 . 00001010
255.255.255.0 = 11111111 . 11111111 . 11111111 . 00000000

11000000 . 10101000 . 00000000 . 00001010
AND
11111111 . 11111111 . 11111111 . 00000000
=
11000000 . 10101000 . 00000001 . 00000000 = 192.168.1.0
```

The last address of the network, known as the **Broadcast Address** is determined by applying the bitwise NOT operation to the subnet mask and then performing a bitwise OR operation with the IP address using the obtained result.

```
192.168.1.10 = 11000000 . 10101000 . 00000000 . 00001010
255.255.255.0 = 11111111 . 11111111 . 11111111 . 00000000

NOT
11111111 . 11111111 . 11111111 . 00000000
=
00000000 . 00000000 . 00000000 . 11111111

-----------------------------------------------------------

00000000 . 00000000 . 00000000 . 11111111
OR
11000000 . 10101000 . 00000000 . 00001010
=
11000000 . 10101000 . 00000000 . 11111111
```

The range of usable IP addresses in this subnet is from 192.168.0.1 to 192.168.0.254. The first (network) and last (broadcasts) addresses are often reserved for network identification and broadcast, respectively so we don't count them as usable hosts.

There you have it! Now you understand what a subnet is and how to calculate it! Let's put this knowledge into practice and build a calculator to handle the job for us!

![Drake and Lil Yachty](https://media.giphy.com/media/800iiDTaNNFOwytONV/giphy.gif)
## Let's build it

Ok, let's start! We're going straight to the nuts and bolts of our subnet calculator. No fuss about looks or design – just the pure functionality, this is what we care about. Let's unravel how it works, step by step.

For our logic, we're gonna use a lightweight Javascript Framework called Alpine.js. Alpine makes everything working with Javascript easier, especially for little projects like this. Using `x-text` (one of Alpine directives) we can set the text content of an element to the result of a given expression. 

To start, let's clarify the information we need and the calculations we want to perform. We'll define expressions for Alpine to either fetch or assign values for specific variables:

From the user, we'll seek two input values:

- IP Address: `ip_address`
- Subnet Mask: `subnet_mask`

Upon completing the calculations, we'll assign the results to the following variables:

- Network Address: `network_address`
- Broadcast Address: `broadcast_address`
- IP Range: `ip_range`
- Usable Hosts: `usable_hosts`
- CIDR Notation: `cidr_notation`
- Wildcard Mask: `wildcard_mask`

After this, we also need an event listener to initialize our calculation process, this can be either a change of the IP Address input or a change of the Subnet Mask input.

```html
<input x-model="ip_address" x-on:input="subnet" ... />
...
<select x-model="subnet_mask" x-on:change="subnet" ... />
	// Subnet mask values
</select>
```

Now each time the input of the IP Address or the Subnet Mask field changes, the `subnet` function gets called.

This kickstarts the whole calculation process. Let's create a diagram of what we want to happen from that moment forward

![Actions Diagram](https://sudorealm.com/post_assets/BWUIphttggrDTPIHCT8VS8MyBcA0TRc0wWHspra1.png)

To put it simply, when a user enters information into the IP Address and Subnet Mask fields, any modification in these fields triggers the `subnet` function. This function, in turn, invokes separate functions for each of the previously defined variables. These dedicated functions are responsible for computing the values assigned to their respective variables.

Now, let's explore the functionality of each of these functions and examine their code.

**Subnetting Function (`subnet`):**
- This function is the main entry point for subnet calculation.
- It calls various functions to compute the network address, broadcast address, IP range, CIDR notation, usable hosts, total hosts, and wildcard mask (For the CIDR Notation value we don't need any extra calculations, just some formatting).

```js
subnet () {
	this.network_address = this.calculateNetworkAddress();
	this.broadcast_address = this.calculateBroadcastAddress();
	this.ip_range = this.calculateRange();
	this.cidr_notation = this.ip_address + "/" + this.subnet_mask;
	this.usable_hosts = this.calculateUsableHosts();
	this.total_hosts = this.calculateTotalHosts();
	this.wildcard_mask = this.calculateWildcardMask();
},
```

**Calculate Network Address (`calculateNetworkAddress`):**    
- Converts the given IP address and subnet mask to binary.
- Applies a bitwise AND operation between the binary representations of the IP address and subnet mask to obtain the network address (we saw that earlier so you already know the procedure 🤓).
- Converts the binary result back to the decimal format and returns it.
```js
calculateNetworkAddress() {
	const ipBinary = this.ipToBinary(this.ip_address);
	const subnetMaskBinary = this.ipToBinary(this.subnet_mask_dec);
	const networkAddressBinary = this.bitwiseAND(ipBinary, subnetMaskBinary);
	return this.binaryToIp(networkAddressBinary);
},
```

**Calculate Broadcast Address (`calculateBroadcastAddress`):**    
- Converts the subnet mask to binary.
- Applies a bitwise NOT operation to the subnet mask.
- Performs a bitwise OR operation between the binary network address and the inverted subnet mask to get the broadcast address.
- Converts the binary result back to the decimal format and returns it.
```js
calculateBroadcastAddress() {
	const subnetMaskBinary = this.ipToBinary(this.subnet_mask_dec);
	const invertedSubnetMaskBinary = this.bitwiseNOT(subnetMaskBinary);
	const broadcastAddressBinary = this.bitwiseOR(this.ipToBinary(this.network_address), invertedSubnetMaskBinary);
	return this.binaryToIp(broadcastAddressBinary);
},
```

**Calculate IP Range (`calculateRange`):**    
- Increments the network address to get the first usable IP.
- Decrements the broadcast address to get the last usable IP.
- Returns a string representing the IP range from the first to the last usable IP.
```js
calculateRange() {
	const firstUsableIp = this.incrementIpAddress()
	const lastUsableIp = this.decrementIpAddress()
	return firstUsableIp + " - " + lastUsableIp;
},
```

**Calculate Usable Hosts (`calculateUsableHosts`):**    
- Determines the total number of addresses in the subnet using `calculateTotalHosts`.
- Subtracts 2 from the total addresses to exclude the network and broadcast addresses.
- Ensures a non-negative result and returns the number of usable hosts.
```js
calculateUsableHosts() {
	const totalAddresses = this.calculateTotalHosts();
	const usableHosts = Math.max(totalAddresses - 2, 0);
	return usableHosts;
},
```

**Calculate Total Hosts (`calculateTotalHosts`):**    
- Determines the total number of addresses in the subnet using the formula $2^{( 32 - subnet mask )}$.
- Returns the total number of hosts, including network and broadcast addresses.
```js
calculateTotalHosts() {
	return Math.pow(2, 32 - this.subnet_mask);
},
```

**Calculate Wildcard Mask (`calculateWildcardMask`):**    
- Converts the subnet mask to binary.
- Applies a bitwise NOT operation to the binary subnet mask to obtain the wildcard mask.
- Converts the binary result back to the decimal format and returns it.
```js
calculateWildcardMask() {
	const binarySubnetMask = this.ipToBinary(this.subnet_mask_dec);
	const binaryWildcardMask = this.bitwiseNOT(binarySubnetMask);
	const wildcardMask = this.binaryToIp(binaryWildcardMask)
	return wildcardMask;
},
```

**Utility Functions (`ipToBinary`, `binaryToIp`, `bitwiseAND`, `bitwiseOR`, `bitwiseNOT`, `incrementIpAddress`, `decrementIpAddress`):**    
- These functions provide utility for converting between binary and decimal representations, performing bitwise AND, OR, and NOT operations, as well as incrementing and decrementing by one IP addresses.

**IP to Binary (`ipToBinary`):**
This function converts an IP address from decimal format to binary format.
- **Input:** `ip` - The IP address in decimal format (e.g., `192.168.1.1`).
- **Output:** The equivalent binary representation of the IP address.
```js
ipToBinary(ip) {
	const octets = ip.split('.');
	const binaryOctets = octets.map(octet => {
		const decimalValue = parseInt(octet);
		const binaryValue = decimalValue.toString(2);
		const paddedBinaryValue = binaryValue.padStart(8, '0');
		return paddedBinaryValue;
	});
	return binaryOctets.join('');
},
```

**Binary to IP (binaryToIp):**
This function converts an IP address from binary format to decimal format.
- **Input:** `binary` - The binary representation of the IP address.
- **Output:** The equivalent IP address in decimal format.
```js
binaryToIp(binary) {
	const binaryParts = binary.match(/.{1,8}/g);
	const decimalOctets = binaryParts.map(part => parseInt(part, 2));
	const ipAddress = decimalOctets.join('.');
	return ipAddress;
},
```

**BitwiseAND (`biwiseAND`):**
This function performs a bitwise AND operation between two binary strings.
- **Input:** `binary1` and `binary2` - Two binary strings of equal length.
- **Output:** The result of the bitwise AND operation.
```js
bitwiseAND(binary1, binary2) {
	const bits1 = binary1.split('');
	const bits2 = binary2.split('');
	const resultBits = [];
	for (let index = 0; index < bits1.length; index++) {
		const resultBit = (bits1[index] === '1' && bits2[index] === '1') ? '1' : '0';
	resultBits.push(resultBit);
	}
	return resultBits.join('');
},
```

**BitwiseOR (`bitwiseOR`):**
This function performs a bitwise OR operation between two binary strings.
- **Input:** `binary1` and `binary2` - Two binary strings of equal length.
- **Output:** The result of the bitwise OR operation.
```js
bitwiseOR(binary1, binary2) {
	const bits1 = binary1.split('');
	const bits2 = binary2.split('');
	const resultBits = [];
	for (let index = 0; index < bits1.length; index++) {
		const resultBit = (bits1[index] === '1' || bits2[index] === '1') ? '1' : '0';
	resultBits.push(resultBit);
	}
	return resultBits.join('');
},
```

**BitwiseOR (`bitwiseOR`):**
This function performs a bitwise NOT operation on a binary string.
- **Input:** `binary` - The binary string to be negated.
- **Output:** The result of the bitwise NOT operation.
```js
bitwiseNOT(binary) {
	const bits = binary.split('');
	const resultBits = [];
	for (let index = 0; index < bits.length; index++) {
		const resultBit = (bits[index] === '1' ? '0' : '1');
		resultBits.push(resultBit);
	}
	return resultBits.join('');
},
```

**Increment IP Address (`incrementIpAddress`):**
This function increments an IP address by 1.
- **Output:** The IP address after incrementing.
```js
incrementIpAddress() {
	const parts = this.network_address.split('.').map(part => parseInt(part));
	for (let i = parts.length - 1; i >= 0; i--) {
		if (parts[i] < 255) {
			parts[i]++;
			break;
		} else {
			parts[i] = 0;
		}
	}
	return parts.join('.');
},
```

**Decrement IP Address (`decrementIpAddress`):**
This function decrements an IP address by 1.
- **Output:** The IP address after decrementing.
```js
decrementIpAddress() {
	const parts = this.broadcast_address.split('.').map(part => parseInt(part));
	for (let i = parts.length - 1; i >= 0; i--) {
		if (parts[i] > 0) {
			parts[i]--;
			break;
		} else {
			parts[i] = 255;
		}
	}
	return parts.join('.');
}
```

## Conclusion
Here you go! This encapsulates the fundamental logic of our calculator. While some additional adjustments may be necessary, they fall outside the scope of this article. For a comprehensive look at the entire source code, feel free to explore [the source code](https://github.com/teobourloglou/subnet-calculator). Thanks for joining me in this project 🙋🏻‍♂️