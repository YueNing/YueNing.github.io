
---
title: Genius User Tutorial
date: 2019-11-06 23:23:47
tags:
	- ANAC
	- Platform
	- Java
	- Tutorial
categories:
	- [Machine learning, Autonomous Negotiation, Tools]
---

## what is Genius

Genius is a Java-based negotiation platform to develop general negotiating agents and create negotiation scenarios. The platform can simulate negotiation sessions and tournaments and provides analytical tools to evaluate the agents' performance

## Installation

<!--more-->

## Requirements

- [Java Runtime Environment 1.8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
- Download the relate version Genius from [here](http://ii.tudelft.nl/genius/?q=article/releases), in this article version is 9.1.11
- Install [IntelliJ](https://www.jetbrains.com/idea/)

  

## Introduction

### How to run
Genius is used in academic research and the international negotiation competition ANAC. In this article IntelliJ is selected to try this Package.

After download Genius, can directly double click file genius-**.jar, open a GUI

After install IntelliJ and configure the system java environment.

### How to develop a new Agent in IntelliJ

1.  Start a new Java project on IntelliJ
2.  Add downloaded genius-9.1.11.jar to the project classpath. In “Idea” you can do that by adding it as a library in Module settings (Figure *Configuration used for developing the Agent*).
3.  Create a new class called SimpleAgent.java and then extend it from the negotiator.Agent class and implement the abstract method called chooseAction(). This method is called whenever our agent needs to perform an action. (i.e.: When the opponent has proposed a bid to the negotiation space, and it’s the turn of our SimpleAgent to either accept it or propose a new bid, then this method will be called.)

<p align="center">
	<img width="400" height="280" src="https://i.loli.net/2019/11/08/OjIv1iJEWnBA2RC.jpg" />
	&nbsp;&nbsp;&nbsp;
	<img width="310" height="280"  src="https://i.loli.net/2019/11/08/D4xwMWqEYXNclvb.jpg" />
</p>

<p align="center">
<em>Configuration used for developing the Agent</em>
</p>

Here is a example Agent created by Malintha Fernando

<details>
	<summary>Simple Agent Example</summary>

```java
import negotiator.*;
import negotiator.actions.Accept;
import negotiator.actions.Action;
import negotiator.actions.EndNegotiation;
import negotiator.actions.Offer;

import java.util.HashMap;
import java.util.Random;
import java.util.logging.Logger;

public class SimpleAgent extends Agent {

    public Action actionOfPartner = null;
    public static Logger log = Logger.getLogger(SimpleAgent.class.getName());
    public Action myLastAction;
    public Bid myLastBid;
    public AgentID agentID;
    public int nOfIssues = 0;
    public HashMap<Bid, Double> utilityCash;

    public enum ActionType {
        START, OFFER, ACCEPT, BREAKOFF
    }

    public ActionType getActionType(Action lAction) {
        ActionType lActionType = ActionType.START;
        if (lAction instanceof Offer)
            lActionType = ActionType.OFFER;
        else if (lAction instanceof Accept)
            lActionType = ActionType.ACCEPT;
        else if (lAction instanceof EndNegotiation)
            lActionType = ActionType.BREAKOFF;
        return lActionType;
    }

    public void init() {
        this.nOfIssues = utilitySpace.getDomain().getIssues().size();
        this.agentID = this.getAgentID();
        this.utilityCash = new HashMap<>();
        BidIterator lIter = new BidIterator(utilitySpace.getDomain());
        while (lIter.hasNext()) {
            Bid tmpBid = lIter.next();
            utilityCash.put(tmpBid, new Double(utilitySpace.getUtility(tmpBid)));
        }
    }

    @Override
    public String getVersion() {
        return "1.0";
    }

    public void ReceiveMessage(Action opponentAction) {
        actionOfPartner = opponentAction;
    }

    @Override
    public Action chooseAction() {
        ActionType lActionType = getActionType(actionOfPartner);
        Action lMyAction = null;
        try {
            switch (lActionType) {
                case OFFER:
                    Offer loffer = ((Offer) actionOfPartner);
                    System.out.println("Opponent's last bid = " + loffer.getBid().toString());
                    if (myLastAction == null) { //this is opponent's initial offer (y0)
                        if (utilitySpace.getUtility(loffer.getBid()) == 1) {
                            lMyAction = new Accept(getAgentID(), loffer.getBid());
                            //i generate my initial offer
                        } else {
                            lMyAction = generateInitialOffer();
                        }
                    } else if (utilitySpace.getUtility(loffer.getBid()) >= utilitySpace.getUtility(((Offer) myLastAction).getBid())) {
                        lMyAction = new Accept(getAgentID(), loffer.getBid());
                        //generate counter offer
                    } else {
                        lMyAction = generateCounterOffer();
                    }
                    break;
                case ACCEPT:
                    break;
                case BREAKOFF:
                    break;
                //I am starting
                default:
                    if (myLastAction == null) {
                        log.info("###default###" + lActionType.name());
                        lMyAction = generateInitialOffer();
                    } else {
                        // simply repeat last action
                        lMyAction = myLastAction;
                        myLastBid = ((Offer) myLastAction).getBid();
                    }
                    break;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        myLastAction = lMyAction;
        return lMyAction;
    }

    public Action generateInitialOffer() throws Exception {
        return new Offer(getAgentID(), getBidRandomWalk());
    }

    public Action generateCounterOffer() throws Exception {
        return new Offer(getAgentID(), getBidRandomWalk());
    }

    public Bid getBidRandomWalk() {
        return utilitySpace.getDomain().getRandomBid(new Random());

    }
} 
```
</details>  


Then compile the project get the SimpleAgent class, open the GUI of Genius, `Java -jar genius-9.1.11.jar` add the new Agent use class file. (Figure Add New Agent)

<p align="center">
	<img src="https://i.loli.net/2019/11/08/7MytNfQ4i8EdsvT.png"/>
</p>

<p align="center">
<em> Add New Agent
</p>

If everything is fine, the can create a negotiation or tournament, use our Agent.

Lucky enough, one of our agents will accept the random bid of the opponent to reach an equilibria.

-------------------------------------------------------------------------------------------
**Reference**

> Genius
> [Official Website][1]

> Genius: An Integrated Environment for Supporting the Design of Generic Automated Negotiators
>[RAZLIN,1SARITKRAUS,1,2TIMBAARSLAG,3DMYTROTYKHONOV,3KOENHINDRIKS,3ANDCATHOLIJNM. JONKER3][2]

> Writing a simple agent in Genius
>[Malintha Fernando][3]

 [1]:http://ii.tudelft.nl/genius/
 [2]:https://homepages.cwi.nl/~baarslag/pub/Genius-An_Integrated_Environment_for_Supporting_the_Design_of_Generic_Automated_Negotiators.pdf
 [3]:https://medium.com/@malintha/writing-a-simple-agent-in-genius-43f13b186e5#.vqafyiqmo
 
> Written with [StackEdit](https://stackedit.io/).