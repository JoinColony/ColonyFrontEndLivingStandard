# Decisions

Decisions is a feature that allows users to use the Governance Extension for proposals that don't interact with the network contracts. 

For example, if I wanted to use the Motions process to decide on which restaurant my team should go to for this Friday's team social, I could create a decision. You can think of it as an "arbitrary" motion, or a motion for anything, not just the standard suite of on-chain interactions. 

## How Decisions Work

The Decisions feature can be broken up into three main views.

1. The Decisions List
2. The Decision Dialog and Preview
3. The Decision details

The Decisions List, available at `/:colony/decisions`, lists all the decisions that have occurred in that Colony. Much like Motions, a Decision requires a 10% stake to appear in the Decisions list. The Decisions list can be filtered by the domain the Decision was created in. Clicking an item in the Decisions list will take the user to the Decision details page, with the exception of the `DraftDecisionItem`, which shows the user if they currently have a draft saved locally.

The Decision Dialog is a rich text input form that allows users to supply their Decision with a title and a description. It will appear when clicking "New Decision". Submitting this form takes the user to a Decision Preview, where they can review, edit, and delete their draft. Once they're happy, they can click "Publish", which will take them to the Decision details page.

Decision details looks mostly the same as for motions, and indeed, a Decision follows the same lifecycle as a motion, with the exception of Finalization (since a Decision doesn't trigger an on-chain action, there's no need to finalize it). The main difference in the UI is the rendering of the decision description as rich text. 

### Decision Drafts

A draft decision is what's initially created when a user submits the Decision Dialog form. This draft needs to be available at `/:colony/decisions/preview`, so a user can preview their draft.

We also need access to it in the Decisions list, since we display a `DraftDecisionItem` there to show the user they have a draft saved locally. 

Drafts are stored in local storage so they persist between page reloads. We also store decision details in Redux, which enables us to re-render any component using these details whenever they change (such as when editing / deleting a draft). When the app loads, we use `ColonyDecisionContext` to take the decision from local storage and populate it in Redux. We then read the decision directly from redux, which gives us access to the decision data globally, which is handy for accessing it from different screens and also from Dialogs. 

### Decision saga

Once a user is happy with their Decision, they can publish it. 

This fires the `MOTION_CREATE_DECISION` redux action, which triggers the `createDecisionMotion` saga.

In this saga, we trigger the `createMotion` action on the `VotingReputation` contract. 

We then store the Decision details in the database, under the `ColonyDecision` model. We store the Decision id in the format: `<colonyAddress>_decision_<txHash>`, so it can be easily retreived from the block ingestor.


### Block ingestor

In the block ingestor, in the same place we handle all the other motion types, we listen for the decision action code (`0x12345678`), and fire our decision handler. Normally, a motion encodes an action to be performed on-chain; in this case, it does not, and so we use a fixed, custom code to communicate the fact we're dealing with a decision.

Inside the Decision handler, all we do is add the decision id to the `ColonyAction` we're creating in the database. DynamoDB will automatically link the `ColonyAction` with its corresponding `ColonyDecision`. We also set the property `isDecision` on the `ColonyMotion` model to `true`. So in this case, the `ColonyAction` will have both `motionData` and `decisionData` fields. Like normal, `motionData` keeps track of the motion as it progresses through its lifecycle, and `decisionData` here points to the `ColonyDecision` created in the saga.

### UI

Once added to the database, this will be picked up in the UI and rendered like a normal motion. There are a couple of differences:

1. There's no "finalizing" a decision, since there's no on-chain action to perform. As a result, we don't include the messages in the message feed referring to finalization, nor do we show the Finalize widget.
2. The decision description, in place of a motion annotation, is rendered as rich text (i.e. html), instead of plain text.

Other than that, a decision goes through the same lifecycle as a motion, and a user can claim their tokens like normal once it finishes. 
