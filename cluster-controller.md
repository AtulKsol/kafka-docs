## Cluster Controller

The controller is one of the Kafka brokers that is also responsible for the task of electing partition leaders (in addition to the usual broker functionality).

**Is the controller just one broker?**

There is only 1 controller at a time.

Going internally, each broker tries to create an ephemeral node in the zookeeper (/controller). The first one succeeds, becoming the controller. The others just get a proper exception ("node already exists"), and watch on the controller node. When the controller dies, the ephemeral node is removed, and the watching brokers are notified. And again, the first one among them which succeeds in registering the ephemeral node, becomes the new controller, the others will once again get the "node already exists" exception and keep on waiting.

**How would you know who is the controller in Kafka?**

When a new controller is elected, it gets a "controller epoch" number by zookeeper. The brokers know the current controller epoch and if they receive a message from a controller with an older number, they know to ignore it.

**Is the controller the leader?**

Not really.. Each partition has its own leader. When a broker dies, the controller goes over all the partitions that need a new leader, determines who the new leader should be (simply a random replica in the in-sync replica list aka ISRs of that partition) and sends a request to all the brokers that contain either the new leaders or the existing followers for those partitions.

The new leaders now know that they need to start serving producer and consumer requests from clients, while the followers now know that they need to start replicating from the new leader.
