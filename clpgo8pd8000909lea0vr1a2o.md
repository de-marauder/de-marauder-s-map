---
title: "The State of the Web - Web5, the new kid on the block"
seoTitle: "Web5 - The State of the web"
seoDescription: "Web 5 is a decentralised peer to peer architecture of the internet implemented to facilitate user centric data control and data portability."
datePublished: Mon Nov 27 2023 08:53:00 GMT+0000 (Coordinated Universal Time)
cuid: clpgo8pd8000909lea0vr1a2o
slug: the-state-of-the-web-web5-the-new-kid-on-the-block
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ZiQkhI7417A/upload/9fb2edd26567eb6638a473133d0b65fb.jpeg
tags: web5, web-50-the-true-decentralization-platform, web5-and-internet

---

The internet has come a long way since its inception in 1983. It hasn’t even been that long and we’ve seen it transition from web1 to web2 to web3 and now web5 is here. In this article, we will be exploring the potential that web5 creates for the internet and its users. LFG!

![](https://lh7-us.googleusercontent.com/qF750EggInmkS93Tq61dzdQvbZk9cEf0PG9fNKY79pcmqbokca-97883sPmarTJawaB54kpaibvS3aCjb-ElF3pQINUCzMqGGuGMHF_9eiDQRethjQwMiMXuc7kRUDUI4JsB41_jpawoTDd0zwnYGD_Swb4bYKjQgNYEWNXDBBLJ2z7F1TDlAy4FPkXffQ align="left")

# Introduction

For a bit of context, let us first have a historical review and get a sense of how it is exactly we got here.

## Web1

The internet in its relatively short lifetime of 40 years has undergone a series of metamorphosis. From web1 where content could only be delivered as static HTML pages styled with CSS. Users could only see static content having no ability to interact with it. There was a lot of room for improvement and the internet did just that. Cue web2!

## Web2

Web2 allowed the internet to share interactive webpages which granted the user the ability to store their data on the websites they visit. This heralded an age of stateful applications that could serve dynamic content based on the user making a request. This boosted the user experience and marketability of websites. Time would go on to reveal a great flaw in this paradigm.

The web2 architecture encouraged the relinquishing of users' data which might be considered private in some cases to be stored in application-specific silos (databases or servers). For example, a company like Google which provides an emailing service would need to store all your very important conversations on their own systems and if for any reason their systems went down or they stopped operation you could potentially lose all the important information you had stored with them. Another great concern was that of privacy and trust. Using websites now requires users to have a great deal of trust in these sites. Trust that their data will always be intact. Trust that it will not be stolen or shared without their permission.

The problem became obvious. The users did not “own” their data so to speak. These websites owned it on their behalf and that was a problem.

The problems with web2 are essentially summarized as the centralization of data. Big websites can be found to have millions of users' data at their disposal. Some take advantage of this problem and make a profit from it by selling user data. Data is not usually portable between similar websites. Websites providing similar services like music players, fintech applications or hospital applications require users to create new identities on them and they store their data in their own site-specific islands. One could argue that technology like OAuth solves the inter-site data sharing problem as it enables the transfer of identity data from one platform to another. However, it is not enough that I can transfer my identity from Google or GitHub to a different application as these applications still hold all of my precious data on their own servers.

These problems with centralization of data birthed web3 with a focus on decentralizing the internet.

## Web3

Web3’s philosophy is centered around openness, transparency, trustlessness, and decentralization. The idea was to replace the need for trust in intermediaries like big tech or finance companies with a distributed ledger hosted on a public blockchain network of interconnected nodes each with a copy of the ledger. The blockchain would be used to keep track of the activities that would occur on the internet. It was designed in such a way that the more activity and users on the blockchain, the harder it would be to make changes to its history. Every user could exist as a node on the chain to validate the credibility of transactions/activities on the chain.

This was a good idea but it had some shortcomings. The nature of the blockchain was such that a single entity could still own the majority of nodes on the chain and as such influence the activities on the chain. If an entity owned 51% of the nodes on a blockchain, it would be able to validate changes to its history. Also, presently, companies like Opensea and Binance facilitate a major chunk of the activity that happens on different blockchains. This would bring us back to the same problem in web2 where users got siloed or trapped into one application.

## Web5

Web5 was created with the sole agenda of giving users back control over their data. It was imagined as the conglomeration of the best parts of web2 and web3. True decentralization of user data would be accomplished and users would not have to go through the hassle of creating multiple copies of their data just to participate in a similar application’s ecosystem. In the place of a blockchain, users could have their own distributed storage where their data is stored and they alone have all rights over the data and can define who else they want to grant permissions to. Users would be able to port their data to different applications by granting these applications to their personal distributed storage. This would just work provided the applications make use of the same schema/protocol.

Unlike web3, web5 does not focus on things such as digital tokens or wallets and NFTs. In fact, there is no use of a blockchain save for some cases where the method for DID (Decentralized Identity) resolution uses a blockchain network. Its only purpose is to give the user back control of their data.

In the following sections, we will discuss just how web5 accomplishes these.

# How it works

Web5 has 3 fundamental pillars, Decentralized Identifiers (DIDs), Verifiable Credentials (VCs), and Decentralized Web Nodes (DWNs). In this section, we will discuss how they work together to provide a decentralized web solution.

[![A slide from TBD’s pitch deck on Web5. | Image: TBD](https://lh7-us.googleusercontent.com/RUoTGYrC0SvrVYIoU8LWsJZ5MFMceLrC5hnVYh5SoZIwPuyb43IzQXVl2gjxaoU0EO6D6U_qVj21Hh86xmMxp83p4v6X4WnZK_YF_IWeAoX4Wc5zoU18NqeFnJSLOsczwpcM35_CZBf1cZOPqSZktCL3yU2qwak2qPddCq6wXbV8yD2gN0KgS04CfW7Rbg align="left")](https://docs.google.com/presentation/d/1SaHGyY9TjPg4a0VNLCsfchoVG1yU3ffTDsPRcU99H1E/edit#slide=id.g13292353b2a_0_0)

## Decentralized Identifiers (DIDs)

A DID is the method of identification in web5. It is a new globally unique identifier who’s specification is recommended by [W3C in the open web standards](https://www.w3.org/TR/did-core/?ref=blog.identity.foundation).

It can be assigned to anything or anyone on a digital network. It replaces the current standard of identification with things such as email addresses or domain names which require centralized management organization for trust reasons. DIDs do not require a centralized authority to maintain their validity. It accomplishes this by granting the owner of the DID cryptographic access to it using public and private keys. DIDs have 4 core properties:

1. **Persistence**: They will always be there and are always unique.
    
2. **Resolvable**: They can be configured to point to other things on the web.
    
3. **Cryptographic verification**: Ownership and validity of the DID can be verified cryptographically.
    
4. **Decentralization**: Since it can be verified cryptographically, it does not require a centralized regulation authority to be trusted.
    

The structure of a DID follows the URN (Uniform Resource Names\]), RFC 8141 schema.

![](https://lh7-us.googleusercontent.com/-7an9Z1RhgSHfYj-3Qw1UmJArnDGIlGUQnVmX_-8hIQAAr__5IjXwpquOGPlC_XPvK-Zwq8-0unuVKeyXzyUkmJKPCCYewmcyM1xWt2Fjnr2_k-XRUZIOnkQzYzk52bZxGMGXaLKROQr7bPH4H4g98BjozQmEfPwE9MbxnBW44KnQ2IS27CyTkoSBhSNGw align="left")

The scheme is always `did` just as a normal web URL scheme would be `http` or `https` for secure connections. The method provides information about how the DID can be resolved. For example, a popular method right now is `ion` which uses the lightning network, a layer 2 bitcoin network to handle DID resolution. Some other methods are: `sov`, `ens`, `web`, `peer`, and `key`.

Only blockchain-based methods like `ion` and `ens` make use of a blockchain for DID resolution. Methods like `web` and `peer` make use of paradigms already prevalent in web2 like HTTP, DNS resolution, and peer-to-peer TCP/IP networks. Right now there are about 32 DID methods in existence and as many as might be required can be created.

For the DID to be useful, it has to be resolved. The resolution method is carried out by a DID resolver. The result of the resolution is the DID document which the DID actually represents. This document is usually a JSON object that contains information about the DID for self-identification. It contains public keys and authentication methods for the DID as well as the signature schema being used. It also contains the service endpoints or web nodes for the DID.

![](https://lh7-us.googleusercontent.com/DkZpbGtnYOR3Ih6ssn4QkAQ0SdwjtjiKKu0Yq-x0lzIcyj0AWaIaipMmBPnidxNzRTp0N9ASJWxgGIdnjFWlTTA-8PpAIIQkpTiwHvk70kRE1ZQ9Nx7r_aNwZzh6XcHuV8iN6BcVEy838jgYeMdtoUwfQzYDMXiTcLWMxLh3rACi6_nTQH37uyHMNccM6w align="left")

The DID method-specific string is a long string that could be just about anything from a web URL to a cryptographic hash. It is what is resolved to the DID document in the same way a domain name resolves to an IP address.

So, the scheme tells us what type of identifier we are working with and the method gives us information on how to resolve the method string.

## Verifiable Credentials (VCs)

Verifiable Credentials are essentially equivalent to certificates. They are also recommended by the W3C open standards. The only difference here between your regular school certificate and a VC is that in this case the VC is signed cryptographically using the DIDs of the issuer and receiver of the certificate. This way, claims made by the receiver can be verified using a VC that supports their claim. It works because of the core properties of DIDs. If every DID is unique and can be made to point to anything on a digital network such as the internet, then an organization like a university can own one and use it to sign VCs for their students using the student’s DIDs. As long as the DID of the university remains unchanged by them, the VC remains valid.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701100580991/e2790003-d961-4583-b3bf-1059cd3b8611.png align="center")

## Decentralized Web Nodes (DWNs)

This is your personal data store. You can think of it as your own database or storage system that holds all your web5 app data. It can be hosted locally on your own devices or remotely on public providers. It is important to note that remote providers will not own your data. They simply have a copy of it and can’t access it if you don’t grant them permission to. You can also seamlessly change providers by adding new service endpoints in your DID doc and removing the old ones once your data has been synced with the new web nodes provided by the service endpoints. The DWNs act like a cluster of replicas continuously sharing data with each other in a process called `sync` to provide redundancy and availability.

# Future Applications

If you’re like me then you’re probably already wondering what you could do with this new technology. The possibilities are endless but I’ll talk about 2 that strike me the most.

## DNS and TCP/IP obsoletion

When I said the DID could be assigned to anything on a digital network I meant it quite literally. Whether is a person or a server. It can be assigned a DID.

![](https://lh7-us.googleusercontent.com/cX1c0N6HQRolXecLgtcjmbFGdQt7U1uzVo1Pcc8I6c1rTmbOVYBpbD6NuXBDzhA7ce-T_L1RLmmWhDqN8S0ZIM5nyS6Qf9tmBFPrh8IYO414AXnt8ExMVTbein4FeMdoS_rwP6_7rI6WUqswrpeOPUndgNH1ZrB0KMfS-Cr-sosoWkBp3VD5CWYRTEwa7g align="left")

This means that the TCP/IP framework for conventional web/internet communication could potentially be rendered obsolete. DNS names could also be affected for the same reason since the methods for resolving DIDs don’t necessarily depend on the present DNS model.

I imagine a scenario where all devices are assigned DIDs and a new protocol is used to facilitate communication between them in a decentralized manner. Your domain names would all be replaced by DIDs which would resolve to web server DIDs which point to DWAs capable of serving web content on demand.

This would cause a radical change in the way computer networking is currently operated. I should add that as we have already indicated earlier, the goal of web5 is not to phase out web2 but rather make it better. This is evident in the existence of a web DID method which allows the conventional TCP/IP model to still function.

## True Serverless

The serverless architecture is one that encourages stateless applications served from ephemeral server nodes to reduce server costs since dormant nodes are terminated and sprung back to life on demand. This architecture is positioned to be the future of applications built on the cloud. In this paradigm, you truly only pay for what you use. No need to keep servers running all day if it’s going to just serve a couple of requests maybe at noon. It saves cost and system resources.

Applications built on web2 are inherently stateful as they have to store user data on centralized servers which must always be available whether it's being used or not. These applications sometimes try but can not ever really be truly serverless.

Applications built on web5 (Decentralized Web Applications \[DWAs or DApps\]) have the potential to be truly serverless. Our discussion has shown that DWAs would be stateless by design since they don’t store any user data but instead make requests to DWNs to fetch required data. The DApp can then make use of this data once they’ve been granted access. The applications need not store any state at all as they can always just request it.

# Concerns

![](https://lh7-us.googleusercontent.com/ZhgBt0vYTQ2YXTgPQGuBB2OTMiPyN3BeFq1JqzjFvZmUvDlGxGVinqMpyrbcPWrbp8gn1sQxop9DUJmbWV-Sw7jWw2CiPE7FbDOnotzowe1jlsELiEkcpKLj2GvMCR7bdAZp0zq4OskppoDYH0JgLkRcmYh9snG4p12mAzKVdb5GnY-bqYTn5SarBYuiVA align="left")

It’s easy to get excited about shiny new tech and this one happens to be particularly shiny. However, there are 3 concerns I can think of.

## Storage costs

The way I see it, the idea of DWNs is essentially encouraging users to set up and manage their own databases or storage solutions and we all know storage especially scalable storage costs a lot. Not everyone will have the privilege and even the technical ability to maintain one. They could decide to opt for remote DWNs, but I can only imagine it will come at its own cost. An argument can be made that infrastructure providers like Google and Amazon can leverage their already existing systems to provide nodes for people and then take access to the public data stored on these nodes as payment. It’s all still sketchy, however. The initial plan was to make sure the user owned their data but this idea seems a bit counterintuitive and the potential of aggregating customer public data might not be viewed as sufficient motivation for vendors like Amazon to provide free DWNs services.

## Scalability issues

In the event that web5 becomes mainstream and is the de facto mode of communication on the Internet. A user could potentially have thousands of DApps and they would all need to query their DWNs for their data. This could pose an issue in cases where all DApps are trying to read or write to the DWNs at the same time. Inconveniences like network lag and more seriously race conditions need to be watched out for and handled properly. This is not the expertise of the typical user so this could slow down the adoption of this remarkable piece of technology.

## DID Recovery

In the event that a user loses the private keys for their DIDs, they are locked out and can no longer perform operations using their DIDs. This is more of a human problem though and surfaces in pretty much any password or key-based system. It is a known problem and experts are continuously ideating on how to solve it.

With all that said, web5 is relatively new and is still being iterated upon. Chances are these concerns will not remain for long.

# But Wait! What about Web 4?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701097105117/f6d1c963-3c7d-428f-b66e-472061c183e6.png align="center")

Well, for starters, it hasn't been forgotten or left out. At least not on an ideological basis. The European Commission has proposed a web4. They herald it as a new internet featuring all the hot technologies of the century. From blockchain to the Internet of Things (IoT) to mixed, virtual, augmented, and extended reality. They define it as, "a blend between artificial intelligence, the internet of things, blockchain, virtual worlds, and extended reality capabilities." This implementation of the web is poised to become potentially greater than web5 as it seems to take anything and everything into consideration. We are not quite there yet, however. The coming years do show promise of numerous technological innovations and breakthroughs indicating that the web4 reality might not be so far off. It is indeed a great time to be alive!

# Conclusion

The internet is continuously morphing in an attempt to serve the needs of its users better. The absence of data privacy and user-centric data control is a very important thing to consider in today’s version of the internet. Web5 exists to solve this problem. Using a combination of DIDs, VCs, and DWAs/DApps users would own their data and have the right to share or hide it from/with external parties.

Web5 heralds a new age of the internet. It would indeed be interesting to see how this plays out. Companies like BlueSky and Zion have already started taking advantage of this technology to build decentralized social media applications with portable user data. You too can leverage this technology and start building DWAs using [TBDs web5 SDKs](https://developer.tbd.website/).

What ideas or concerns do you have about web5? Share them in the comment section. I’d love to get your opinion on the matter.