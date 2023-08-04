# Motions

Motions allow users to collectively approve or reject an action in a Colony. Whereas a normal action is unilateral--i.e. it can be completed from start to finish synchronously by a single user--motions provide others an opportunity to review, and if desired, object to, the proposed action.

## How Motions Work

### The Governance Extension

Underpinning the Motions feature is the `VotingReputation` contract. Motions functionality can be enabled in a Colony by installing the Voting Reputation (Governance) extension. The extension takes a number of initialisation parameters, which determine things like how long the motion phases lasts for, and what the staking and voting thresholds are. More on that later.

Once the Governance extension has been installed, so long as there is some reputation in the Colony, a user can create a motion in the Colony. They do not need to be joined to the Colony in order to be able to create a motion in it. 

### The Motion Lifecycle

A motion goes through a number of phases from creation to completion. The Staking, Voting, and Reveal phases each have a duration, which is specified when installing the Governance extension. If the phase has not already transitioned to a subsequent phase by the end of the timer, it will do so automatically.

Let's take a look at these phases in a little more detail:

1. Creation. First, a motion gets created. Once the Governance extension is installed, this becomes the default method for triggering an action, and can only be overridden if the user has the correct permissions. 

2. Staking. Staking means to put up some tokens in support of the motion, which will be returned to you in full if the motion passes, and possibly with a penalty applied if it doesn't.

In order to pass, a motion needs to be staked by 100% of the **required stake**.The amount that represents "100%" here is determined when initialising the Governance Extension. It is calculated as a percentage of the domain's reputation, in token terms. 

So for example, if the motion is being made in the root domain, and the root domain has `1_000_000` reputation, a required stake of 1% would require that a motion need be staked by `10_000` of the Colony's native token in order to pass (in WEI). 1 token = 1 * 10^18 WEI, which means `10_000` WEI is actually 1 * 10^-14 token.

Anybody can stake a motion, not just its creator. When staking a motion, there are a few constraints applied to each user:

- A user must have a minimum amount of reputation in the domain the motion was created in. This is calculated as a percentage of the required stake, and the precise percentage is determined when installing the Governance extension.
- A user must also have this same minimum in token terms as well. 

So for example, let's say the required stake is `10_000` WEI, and the minimum stake % is 1%.
That means the user's minimum stake is `100` WEI. 

To stake, a user must have `100` reputation AND `100` WEI of the Colony's native token. 

- In addition, a user cannot stake more tokens than they have reputation (their "max stake"). If a user's reputation is less than the total amount left to stake, the user will not be able to stake the motion fully, even if they have enough tokens to be able to do so. 

So far, I've been referring to staking in the positive direction, or "for" a motion. It's also possible to stake "against" a motion, which we refer to as an "objection". An objection has the same threshold in order to be considered valid. 

If a motion is not staked (positively) by 100% of the required stake, the motion will fail. 
If a motion is staked 100%, and does not have an objection that is also staked 100%, the motion will pass. 

These are the two simplest states of a motion outcome. Direct pass, and direct fail.

In either of these cases, a motion will progress directly to the Finalize phase if it passed, or to the Claim phase if it failed. 

The third case, in which a motion is both staked and objected 100%, triggers a Vote on the motion. In this case, the motion will immediately progress to the Voting phase, and not wait for the staking phase duration to elapse. 

3. Voting. A vote is open to any wallet address with more than 0 reputation in the colony. You can either vote for the motion or against it. While still in the Voting phase, a user can change their vote as many times as they like. The voting phase will transition to the Reveal phase either when the Voting duration has elapsed, or when a certain minimum amount of reputation has voted on the motion (also specified when installing the Governance extension). That is to say, when the combined reputation of all the users that have voted on the motion reaches a certain threshold, the Voting phase may pass to Reveal without waiting for the duration to elapse.

4. Reveal. This is where users reveal their votes to one another. Initially votes are hidden, and only in the reveal phase do users share them publicly. This is where the outcome of the motion is determined. Once all voters have revealed their votes, the side with the greatest number of votes wins, and the motion progresses to the Finalization phase. If not all voters reveal their votes, the motion will progress regardless once the Reveal phase duration has elapsed.

5. Finalization. Anybody can finalize a motion, and it can only occur once. The result here depends on whether the motion passed or failed. If the motion failed, then as far as the users are concerned, nothing happens when a motion is finalized. However, if the motion passed, then the underlying on-chain action will be executed. Once a motion is finalized, it transitions to the Claim phase.

6. The Claim phase is where anybody who staked the motion can have their stake returned. This applies whether the motion went to a vote or not. In the event a motion went to a vote, those on the winning side will receive a reward above their initial stake, which comes from the penalty applied to the stakes of those on the losing side. This incentivizes people to only stake / object motions that matter to them!


### The UI

Now that we have an understanding of the various phases a motion can be in, we can discuss how we present this information to users. 

#### Motion Details page
We reuse the Action details page layout for motions. 

When viewing the motion details page, a user will be able to see which phase a motion is in at any given time. They will be able to interact with the motion via the `MotionPhaseWidget`, and they will see the messages in the message feed, which is essentially of a log of the motion's lifecycle history. 

#### Actions list

Motions show up in the actions list, just like ordinary actions, with one caveat. Motions need to be staked a minimum of 10% of the minimum stake in either direction.

The key thing to be aware of here is that there are **two** sources of data mutations on the `ColonyMotion` object. The `ColonyMotion` model is what we use to keep track of all the information relevant to a motion (which messages it shows, who's staked, who's voted etc.). This can be mutated via the block ingestor, or via our `fetchMotionState` lambda. 

### `fetchMotionState` lambda

This lambda has a few responsibilities. 

1. It will return you the correct phase of the motion. We don't store this data on the `ColonyMotion`, because it can only be determined by querying the `VotingReputation` contract. This is because motion state changes are not purely event-driven. A motion can also change state if the phase duration elapses, and in this case, an event will not be emitted by the contract. Thus, to be sure of which phase our motion is in, we need to query the contract directly, and we do so in this lambda.

2. As a result of knowing which phase the motion is in, we need to update the message feed to include the corresponding message. This cannot be done in the block ingestor, for the same reason as above: not all motion state changes are triggered by user activity.  

3. Depending on which phase our motion is in, we may have to perform additional calculations such as calculating staker rewards. For instance, if a motion is staked, but doesn't go to a vote, we need to be able to return the stakers their stake once it finishes. In this case, it will finish when the staking duration elapses, and so again, this can't be calculated in the block-ingestor. 

In summary, this lambda fetches the latest motion state from the chain, and if necessary, performs mutations to the corresponding `ColonyMotion` database entry to ensure it correctly reflects the current state.

Note: "state" here is being used ambiguously. On the one hand, it refers to the phase a motion is in. On the other, it refers to all of the data we need to keep track of in order to display it to the user in the UI.

### Block ingestor

The block ingestor will also mutate the `ColonyMotion` object for a given motion. As indicated above, these mutations are event-driven. Whenever a user interacts with a motion, we can pick up the corresponding event in the block-ingestor and handle it there. 

Specifically, these include: 

- When a motion is created
- When a motion is staked
- When a motion is voted on
- When a vote is revealed
- When a motion is finalized
- When a motion reward is claimed

For each event, we will store its data on the `ColonyMotion` database entry, so that users can always see the latest motion data in the UI. 

We handle most motion events uniformly, with the exception of `MotionCreated`. When a motion is created, it encodes the action that will be executed if it passes. Therefore, depending on which action is encoded, we will store different data accordingly. 

### Database

We have a `ColonyMotion` model for all data directly relevant to a motion. 

This is accessible from the parent `ColonyAction` model. Since a motion ultimately performs an on-chain action in the event it's successful, we consider it subordinate to the `ColonyAction`. We also do this because motions and actions are accessible via the same route; therefore, this pattern lets us know what to query from the database (a `ColonyAction`), and then be able to render an action view or a motion view depending on whether the `ColonyAction` contains motion data. 

## Summary

Motions is a complex feature. 

For further technical specifics, see the `VotingReputation` contract in the `colonyNetwork` repo, or for a higher-level, user-oriented overview, see the [Motions](https://docs.colony.io/learn/governance/motions/) section of the Colony docs. 








