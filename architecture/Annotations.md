# Annotations

An annotation is a string that can be used to provide context to an action or motion. 

For example, if I want to mint some tokens, I might write an annotation to explain why, so other colony members can better understand the rationale behind my decision.

## How annotations work

### User land

Annotations are captured in the UI via text inputs. There are two ways a user can make an annotation.

First, when creating an action or motion.

Second, when objecting to a motion.

After confirming, the annotation will be passed to the corresponding saga as part of the payload. 

### Saga 

In the saga, we do three things. 

First, we upload the annotation to IPFS. This means it can be accessed even if the user isn't accessing Colony via the CDapp.

Then, we upload it to the database using the [Annotation model](https://github.com/JoinColony/colonyCDapp/blob/7c0177dddb4d2809de096c1df0feb140ae8849f8/amplify/backend/api/colonycdapp/schema.graphql#L2581). The model has four fields:

`id`: the transaction hash of the transaction it annotates, 
`message`: the annotation message, 
`ipfsHash`: the annotation's ipfs hash 
`actionId`: the id of the action the annotation is associated with. Generally, this will be the same as the annotation id, with the exception of an objection to a motion. Since that annotation actually annotates the `stakeMotion` transaction, not the original action / motion transaction, we store the `actionId` separately.

Finally, we call the `annotateTransaction` method on the Colony contract. This emits an `Annotation` event we are listening for in the block ingestor. The event has three params:

`msgSender`: the wallet address making the annotation
`txHash`: the hash of the transaction it's annotating
`metadata`: an arbitrary string, which we use to store the ipfs hash

### Block ingestor 

In the block ingestor, we're only concerned with the `txHash`. We use the `txHash` to query the annotation in the database (since this is the annotation's id). From there, we look at the `actionId`. 

If the `txHash` and the `actionId` are identical, we know we're not objecting to a motion, and so we write the `txHash` to the `annotationId` field on the corresponding [`ColonyAction`](https://github.com/JoinColony/colonyCDapp/blob/7c0177dddb4d2809de096c1df0feb140ae8849f8/amplify/backend/api/colonycdapp/schema.graphql#L2147) entry. Behind the scenes, DynamoDB will associate the Annotation with the `ColonyAction`, so we can access it via the UI.

If they're different, we know the annotation is for an objection to a motion. In this case, we retreive the motion, and store the `annotationId` on the corresponding `ColonyMotion`. Since objection annotations can only be performed on motions, it makes sense that an objection annotation should be associated with the `ColonyMotion` model. Again, DynamoDB allows us to access the Annotation from the `ColonyMotion` we associate it with, for consumption in the UI.

### UI

In the UI, we have an `ActionAnnotation` component, which renders the annotation for the user when they visit the Action / Motion details page. It has a different background colour depending on whether the annotation is of the original action, or an objection to a motion. 

