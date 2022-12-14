High Level Approach:
In order to complete this assignment we followed the RAFT protocol and implemented all the necessary safety measures and election processes:

Starting with the initial states: Each replica has following fields:
State: for the current state they are in, whether they are in follower, candidate, or leader state. Each replica starts off as follower.
Term: the term number of the replica and each replica starts off with term 0
TransactionLog: The log/statemachine of the replica
Msgtimer: A timestamp of the last message received from the leader
Timeout: randomly generated timeout for determining whether the leader has failed and there needs to be a new election

Committed: int for determining how many entries have reached quorum from the group of nodes.
LastIndex: Index that corresponds to where the leader believes the replicas log index is.
ResponseLen: Index that corresponds to how many messages have been acknowledged by the replica.
Lastly fields for determining how many votes a replica has received and who they have votedFor in the current term.

Walking you through the program, starting with the run method, we first have two cases one for whether the state of the replica is either a candidate or follower, and the other for when the state is the leader. If we are in the follower state/candidate state we have a timer that determines based on our randomly selected timeout whether we should start a new election(following the 5.2 procedure and 5.4.1 safety procedure). If we are in the leader state we have a timer for sending heartbeats to the other followers.

While in the run method we listen for requests from the client, when we receive a request in the follower/candidate state and we receive a get/put request from a client we store that message in a list and once we know the leader, we then redirect the client to that leader. Once the leader receives that request, we then store it within our transaction log and then forward that request along with the other necessary log entries, to the rest of the other replicas.

Once the replica receives that message it determines based on the 5.3 Log replication rules, whether it is safe to append the entry onto its log, and once it does it sends back a response to the leader that it has acknowledged the messages, if the log cannot it sends a fail message for the leader to decrease its index by 1 to try again.

Once the leader gets back enough ackowlegdments from the majority of the replicas, it can safely commit the log entry, to its kvstore, and sends back an ok message to the client, and the process then continues for each of the put requests that the client sends.

Challenges Faced:
Committing on both leader and follower: this required logs to be perfectly up to date (not missing entries and consistent with each other especially after partitions or crashes) however with other methods, the logs at times were inconsistent and created inconsistent kv stores when I tried to commit.

Election timeouts/redirects: The challenges here were first trying to find a way to detect when timeouts happened with the leader, and then keeping the connection alive with the leaders and its replicas to not continuously timeout, and lastly maintain the election safety mechanisms with elections and term numbers.

List of Properties/Features of design that we feel is good:
First we added a while loop for when our leader commits messages, it allows us to respond back to the client much faster with entries we know that are committed and reduces Median Response Latency.

Log Replication, we efficiently remove any entries that have come from a lesser/previous term and append onto the replicas transaction log with the entries that they only need.

Overview of how we tested our code:
Our main form of testing was running it through the simulator and littering our code with lots of print statements, since the advanced tests built on top of the previous tests, we did not move onto more challenging and difficult tests until we passed the lesser ones(i.e partitions and crashes) comfortably and made sure the output was expected and was reasonable and passed the previous tests well.

