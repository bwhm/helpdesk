# Table of contents
<!-- TOC START min:1 max:3 link:true asterisk:false update:true -->
- [Table of contents](#table-of-contents)
- [Overview](#overview)
- [Contribute](#contribute)
- [Extrinsics](#extrinsics)
  - [balances](#balances)
  - [consensus](#consensus)
  - [council](#council)
  - [councilElection](#councilelection)
  - [dataDirectory](#datadirectory)
  - [dataObjectStorageRegistry](#dataobjectstorageregistry)
  - [dataObjectTypeRegistry](#dataobjecttyperegistry)
  - [downloads](#downloads)
  - [grandpaFinality](#grandpafinality)
  - [election](#election)
  - [finalityTracker](#finalitytracker)
  - [grandpa](#grandpa)
  - [members](#members)
  - [memo](#memo)
  - [proposals](#proposals)
  - [session](#session)
  - [staking](#staking)
  - [sudo](#sudo)
  - [timestamp](#timestamp)
- [Chain State](#chain-state)
  - [Definitions](#definitions)
  - [actors](#actors)
  - [balances](#balances-1)
  - [consensus](#consensus-1)
  - [council](#council-1)
  - [councilElection](#councilelection-1)
  - [dataDirectory](#datadirectory-1)
  - [dataObjectStorageRegistry](#dataobjectstorageregistry-1)
  - [dataObjectTypeRegistry](#dataobjecttyperegistry-1)
  - [downloadSessions](#downloadsessions)
  - [grandpaFinality](#grandpafinality-1)
  - [indices](#indices)
  - [membership](#membership)
  - [memo](#memo-1)
  - [migration](#migration)
  - [proposals](#proposals-1)
  - [session](#session-1)
  - [staking](#staking-1)
  - [substrate](#substrate)
  - [sudo](#sudo-1)
  - [system](#system)
  - [timestamp](#timestamp-1)
- [Toolbox](#toolbox)
  - [RPC calls](#rpc-calls)
  - [Hash data](#hash-data)
  - [Sign message](#sign-message)
  - [Verify signature](#verify-signature)
<!-- TOC END -->

# Overview
This page contains information on advanced [transactions](#extrinsics), called `extrinsics` and [`chain state`](#chain-state) queries. Note that this is a WIP, so more information will be added continuously.

# Contribute
This page is hard to properly document and maintain. All contributions are welcome, and will qualify for [bounties](../#builders-and-bug-reporters).

# Extrinsics

Extrinsics are a general term for transactions, as the latter implies the more specific transfer of tokens from A to B.

## balances

## consensus

## council

## councilElection

## dataDirectory

## dataObjectStorageRegistry

## dataObjectTypeRegistry

## downloads

## grandpaFinality

## election

## finalityTracker

## grandpa

## members

## memo

## proposals

## session

## staking

## sudo

## timestamp


# Chain State
The `Chain State` sidebar allows you to query for information regarding the current state of the chain.

## Definitions

##### `AccountID`
Equivalent to `address`.

##### `Actor`
The `AccountID` associated with a [role](#role).

##### `balanceOf`
The staking reward for `Validators`, (ie. the change in balance).

##### `blockNumber`
Number of blocks. Sometimes conflated with [`eras`](#bondingDuration).

##### `MemberID`
An integer corresponding to the order of which the `AccountID` signed up for a [membership](#membership

##### `Perbill`
An integer representing a fraction per billion.

##### `Role`
A role on the platform that requires a [membership](#membership), and [staking](#staking).

##### `u32`
An integer. E.g. the number of `Validators`.

##### `u64`
An integer. E.g. the number of blocks.


## actors
All information regarding the state and parameters for `roles` and `actors` can be queried here.

Note that at this stage, the only [role](#role) and `actor` can take is [Storage Provider](../roles/storage-provider).

#### accountIdsByMemberId
* Explicit: `accountIdsByMemberId(MemberId): Vec<AccountId>`
* Explanation: `actor accounts associated with a member id`

Argument: [`MemberId`](#memberid)
Returns: List of the [`AccountIDs`](#accountid) of each [`role`](#role) associated with the [member](#membership).

Note: It will not show the [`AccountID`](#accountid) of the [`member`](#membership), rather the `AccountID` generated when signing up for a `role`. This is linked to the `MemberID`.


#### accountIdsByRole
* Explicit: `accountIdsByRole(Role): Vec<AccountId>`
* Explanation: `actor accounts associated with a role`

Argument: [`Role`](#role)
Returns: List of the [`AccountIDs`](#accountid) of each [`role`](#role) associated with the [member](#membership).

Note: The `role` input must be in `hex` form:
* [Storage Provider](../roles/storage-provider) = `0x00`

#### actorAccountIds
* Explicit: `actorAccountIds(): Vec<AccountId>`
* Explanation: `Actors list`

Argument: `none`
Returns: List of the [`AccountIDs`](#accountid) for all [`roles`](#role) .

Note:

#### actorByAccountId
* Explicit: `actorByAccountId(AccountId): Option<Actor>`
* Explanation: `actor accounts mapped to their actor`

Argument: [`AccountID`](#accountid)
Returns: A JSON in the following format:
```
{"member_id":<n>,"role":<role_id>,"account":"<AccountID>","joined_at":<blockNumber>}
```
Note:
* `<n>` is `MemberID` (integer) in chronological order of when the `member` signed up for a `membership`.
* `<role_id>` is the integer assigned to the role the `AccountID` occupies.
* `<blockNumber` is the block of which the `AccountID` got accepted in to the `role`.

#### availableRoles
* Explicit: `availableRoles(): Vec<Role>`
* Explanation: `the roles members can enter into`

Argument: `none`
Returns: The name of `role(s)` that can be occupied on the platform.

Note:

#### parameters
* Explicit: `parameters(Role): Option<RoleParameters>`
* Explanation: `requirements to enter and maintain status in roles`

Argument: The `hex` ID of `role` that is being queried.
Returns: A JSON in the following format:
```
{"min_stake":<min_tokens>,"min_actors":<min_actors>,"max_actors":<max_actors>,"reward":<reward>,"reward_period":<reward_period>,"bonding_period":<bonding_period>,"unbonding_period":<unbonding_period>,"min_service_period":<min_service_period>,"startup_grace_period":<startup_grace_period>,"entry_request_fee":<fee>}
```
Note:
* `<min_tokens>` the minimum stake required for the `role`
* `<min_actors>` the minimum number of `actors` required for the service to be provided.
* `<max_actors>` the maximum number of `actors` that can occupy the `role`.
* `<reward>` the reward in Joy for occupying a `role`.
* `<reward_period>` the frequency of payouts in `blocks`.
* `<bonding_period>` the number `blocks` after confirming the request before you actually occupy the role.
* `<unbonding_period>` the number of `blocks` from the holder of the `role` `unstake`, to the tokens are `unbonded` and spendable.
* `<min_service_period>` the minimum number of `blocks` you must occupy the `role` before you can `unstake`.
* `<startup_grace_period>` amount of `blocks` from singing up for the `role` before you get offically "join", ref [actorByAccountId](#actorbyaccountid).
* `<fee>` cost in Joy for signing up. Unlike the `stake`, this is non-refundable and burned.


Note:

#### requestLifeTime
* Explicit: `requestLifeTime(): u64`
* Explanation: `Entry request expires after this number of blocks`

Argument: `none`
Returns: An integer representing the number of `blocks` from a potential `actor` applies for a `role` until the request expires.

Note:

#### roleEntryRequests
* Explicit: `roleEntryRequests(): Requests`
* Explanation: `First step before enter a role is registering intent with a new account/key.`

Argument: `none`
Returns: A list in the following format:

```
# If there is one or more requests
[["<AccountID(actor)>",<member_id>,<role_id>,<blockNumber>],[...]]
# If there are no requests:
[]
```

Note:
* `<AccountID(actor)>` is the address of the `actor` requesting the `role`
* `<member_id>` is `MemberID` (integer) of `member` associated with the `actor` `AccountID`.
* `<role_id>` is the id (integer) of the `role`.
* `<blockNumber>` is the block heigh of which the request was made.


## balances

## consensus

## council

## councilElection

## dataDirectory

## dataObjectStorageRegistry

## dataObjectTypeRegistry

## downloadSessions

## grandpaFinality

## indices

## membership

## memo

## migration

## proposals

## session

## staking
All information regarding the state and parameters for `Validators`, `nominators` and general information about block production can be queried here.

#### bonded
* Explicit: `bonded(AccountId): Option<AccountId>`
* Explanation: `Map from all locked "stash" accounts to the controller account.`

Argument: `AccountID` (`stash`)
Returns: The `AccountID`(`controller`) mapped to the `stash`.

Note: -

#### bondingDuration
* Explicit: `bondingDuration(): BlockNumber`
* Explanation: `The length of the bonding duration in blocks.`

Argument: `none`
Returns: The number of `eras` before the tokens that are being [unbonded](#unbond) are free to [withdraw](#withdrawunbonded).

Note: Although the explanation says blocks, the duration is actually in denominated in `eras` ref [this](https://github.com/paritytech/substrate/pull/2289) issue.

#### currentElected
* Explicit: `currentElected(): Vec<AccountId>`
* Explanation: `The currently elected validator set keyed by stash account ID.`

Argument: `none`
Returns: A list of the `AccountID` (`stash`) of all current `Validators`.

Note:

#### currentEra
* Explicit: `currentEra(): BlockNumber`
* Explanation: `The current era index.`

Argument: `none`
Returns: The current `era`.

Note: Although each `era` lasts 600 blocks, certain events will trigger new `eras`. As a consequence, `blockNumber`/600 may show a lower number `currentEra`.

#### currentEraReward
* Explicit: `currentEraReward(): BalanceOf`
* Explanation: `The accumulated reward for the current era. Reset to zero at the beginning of the era`

Argument: `none`
Returns: The staking reward payed out each `era` in Joy.

Note:

#### currentSessionReward
* Explicit: `currentSessionReward(): BalanceOf`
* Explanation: `Maximum reward, per validator, that is provided per acceptable session.`

Argument: `none`
Returns: Deprecated.

Note: Deprecated.

#### forcingNewEra
* Explicit: `forcingNewEra(): Option<Null>`
* Explanation: `We are forcing a new era.`

Argument: `none`
Returns: `<empty>` unless query is made on the first block of a new `era`. Then returns _fix_

Note:

#### invulnerables
* Explicit: `invulnerables(): Vec<AccountId>`
* Explanation: `Any validators that may never be slashed or forcibly kicked. It's a Vec since they're easy to initialize`

Argument: `none`
Returns: The `AccountIDs`(`controller`) that can't be slashed, if any.

Note:

#### lastEraLengthChange
* Explicit: `lastEraLengthChange(): BlockNumber`
* Explanation: `The session index at which the era length last changed.`

Argument: `none`
Returns: The `session` index when the `council` or `sudo` key changed the length of each `era`.

Note:

#### ledger
* Explicit: `ledger(AccountId): Option<StakingLedger>`
* Explanation: `Map from all (unlocked) "controller" accounts to the info regarding the staking.`

Argument: `AccountID`(`controller`)
Returns: A JSON in the following format:
```
{"stash":"<AccountID(stash)>","total":<tot_bonded>,"active":,<act_bonded>:[{"value":<unbonding>,"era":<era>}]}
```
Note:
* `<AccountID(stash)>` is the address of the `stash` linked to the `controller`
* `<tot_bonded>` Is the total amount you have staked/bonded
* `<act_bonded>` Is the amount of tokens that is not being unlocked
* `<unbonding>` Is the amount of tokens that is in the process of being freed
  * `<unbonding>` + `<act_bonded>` = `<tot_bonded>`
* `<era>` Is the `era` when your tokens will be "free" to transfer/bond/vote
  * See [currentEra](#currentEra) to make sense of `<era>`.


#### minimumValidatorCount
* Explicit: `minimumValidatorCount(): u32`
* Explanation: `Minimum number of staking participants before emergency conditions are imposed.`

Argument: `none`
Returns: The minimum number of `Validators` required for blocks to be produced.

Note:

#### nextSessionsPerEra
* Explicit: `nextSessionsPerEra(): Option<BlockNumber>`
* Explanation: `The next value of sessions per era.`

Argument: `none`
Returns: `<empty>` unless query is made on the first block of a new `era`. Then returns _fix_

Note:

#### nominators
* Explicit: `nominators(AccountId): (Vec<AccountId>, Linkage<AccountId>)`
* Explanation: `The map from nominator stash key to the set of stash keys of all validators to nominate.`

Argument: `AccountID`(`stash`) of a `Nominator`
Returns: A JSON in the following format:
```
[["<AccountID(stash)-of-validator>"],{"previous":"<AccountID(stash)-of-nominator>","next":"<AccountID(stash)-of-nominator>"}]
```

Note:
Assuming the `Validator` has three `nominators`. `nominator_1`, `nominator_2` and `nominator_3` where the index denotes the order of when they started nominating.
```
# If you make the query with nominator_1, it will show:
{"previous":"<nominator_2>","next":"null"}
# If you make the query with nominator_2, it will show:
{"previous":"<nominator_3>","next":"<nominator_1>"}
# If you make the query with nominator_3, it will show:
{"previous":"null","next":"<nominator_2>"}
```

#### offlineSlash
* Explicit: `offlineSlash(): Perbill`
* Explanation: `Slash, per validator that is taken for the first time they are found to be offline.`

Argument: `none`
Returns: The amount slashed [`perbill`](#perbill) if the a `Validator` has been reported offline more than the [offlineSlashGrace](#offlineslashgrace) allows.

Note:
* If the `Validator` has bonded 1000Joy, and the [`perbill`](#perbill) amount is `10000000`, then the `Validator` is slashed:
```
1000Joy * (10000000/1*10^9) = 10Joy
```
* As soon as you exceed the [offlineSlashGrace](#offlineslashgrace) (`<n>`), the first slash will incurr retroactive slashing such that you are actually slashed:
```
1000Joy * (10000000/1*10^9) * <n> = <n>*10Joy
```

#### offlineSlashGrace
* Explicit: `offlineSlashGrace(): u32`
* Explanation: `Number of instances of offline reports before slashing begins for validators.`

Argument: `none`
Returns: The number of times a `Validator` can be reported offline before getting [slashed](#offlineslash).

Note:


#### payee
* Explicit: `payee(AccountId): RewardDestination`
* Explanation: `Where the reward payment should be made. Keyed by stash.`

Argument: `AccountID`(`stash`)
Returns: How the `Validator` has configured his [bonding preferences](../roles/validators#bonding-preferences) payout settings.

Note: Options are:
* `Staked` - the funds are transferred to the `Validator`(`stash`) and added to the bonded tokens.
* `Stash` - the funds are transferred to the `Validator`(`stash`), but not added to the bonded tokens.
* `Controller` - the funds are transferred to the `Validator`(`controller`).


#### recentlyOffline
* Explicit: `recentlyOffline(): Vec<(AccountId,BlockNumber,u32)>`
* Explanation: `Most recent 'RECENT_OFFLINE_COUNT' instances. (who it was, when it was reported, how many instances they were offline for).`

Argument: `none`
Returns: A list in the following format:
```
[["<AccountID(stash)>",<blockNumber>,<number_of_warnings>],["AccountID(stash)",<blockNumber>,<number_of_warnings>],...["AccountID(stash)",<blockNumber>,<number_of_warnings>]]
```
Note: `<number_of_warnings>` will always be `1` as you are only given one warning per offline reporting.

#### sessionReward
* Explicit: `sessionReward(): Perbill`
* Explanation: `Maximum reward, per validator, that is provided per acceptable session.`

Argument: `none`
Returns: The maximum reward per `Validator` [perbill](#perbill) per acceptable session.

Note: If the `Validator` with the [lowest bonded tokens](#slotstake) has bonded 1000Joy, and the [`perbill`](#perbill) amount is `1000000`, then all `Validator` earns (per session):
```
1000Joy * (1000000/1*10^9) = 1Joy
```

#### sessionsPerEra
* Explicit: `sessionsPerEra(): BlockNumber`
* Explanation: `The`

Argument: `none`
Returns: The number of `sessions` per `era`.

Note:

#### slashCount
* Explicit: `slashCount(AccountId): u32`
* Explanation: `The number of times a given validator has been reported offline. This gets decremented by one each era that passes.`

Argument: `AccountID`(`stash`)
Returns: An integer representing the number of times the `Validator` has been reported offline.

Note: If this number is higher than the [offline slash grace](#offlineSlashGrace), the `Validator` will get [slashed](#offlineslash) next time it gets reported.


#### slotStake
* Explicit: `slotStake(): BalanceOf`
* Explanation: `The amount of balance actively at stake for each validator slot, currently.`

Argument: `none`
Returns: The bonded stake (in Joy) of the `Validator` with the least amount of tokens bonded.

Note:


#### stakers
* Explicit: `stakers(AccountId): Exposure`
* Explanation: `Nominators for a particular account that is in action right now. You can't iterate through validators here,`

Argument: `AccountID`(`stash`)
Returns: A JSON in the following format:
```
# If the Validator has no Nominators:
{"total":<total_validator_stake>,"own":<validators_stake>,"others":[]}

# If the Validator has Nominators:
{"total":<total_validator_stake>,"own":<validators_stake>,"others":[{"who":"<AccountID_of_nominator_1(stash)>","value":<stake_of_nominator_1>},....,{"who":"<AccountID_of_nominator_n(stash)","value":<stake_of_nominator_n>}]}
```

Note:

#### validatorCount
* Explicit: `validatorCount(): u32`
* Explanation: `The ideal number of staking participants.`

Argument: `none`
Returns: An integer representing the maximum amount of active `Validators`.

Note:

#### validators
* Explicit: `validators(AccountId): (ValidatorPrefs, Linkage<AccountId>)`
* Explanation: `The map from (wannabe) validator stash key to the preferences of that validator.`

Argument: `AccountID`(`stash`)
Returns: A JSON in the following format:

```
[{"unstakeThreshold":<n>,"validatorPayment":<token>},{"previous":"<AccountID(stash)_1>","next":"<AccountID(stash)>_2"}]
```

Note:
* `<n>` is the number of times the `Validator` can be slashed for being offline before it automatically `unstake` on the next time it gets reported offline. This is set when you first start [`validating`](../roles/validators#configure-your-validator-keys), default [validating preferences](../roles/validators#validating-preferences) is `3`.
  * It should be added that if your `validator` node goes offline, you will be [slashed](#offlineslash) `<n>`+ [`grace`](#offlineslashgrace) + 1 times. As you accept `<n>`, `<n+1>` triggers `unstake`.
* `<token>` means the number of tokens the `Validator` claims for themselves before sharing the rest with potential `nominators` based on the share of [stake](#stakers).
* `<AccountID(stash)_1>` is the last `Validator` that got accepted before you. (or `null`)
* `<AccountID(stash)_1>` is the next `Validator` in the queue. (or `null`)


## substrate

## sudo

## system

## timestamp

# Toolbox

## RPC calls

## Hash data

## Sign message

## Verify signature
