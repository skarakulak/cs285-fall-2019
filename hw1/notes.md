* Define a module in TF:
  
    * A function that returns placeholders
        * Input: 
        * Ground truth
    * A forward:
        * That takes the placeholder and output the prediction place holder
    * A `train_op`
        * That takes all placeholders
        * That uses forward
        * Compute loss
        * Return a optimization OP


* 5-tuple
  * `o`: observation, ndarray of some shape
  * `a`: action, ndarray of some shape
  * `o_next`
  * `r`: float number
  * `t`: bool
  
* path: an dictionary containing episode info.
  
* Replaybuffer:
    * A very large FIFO queue of sequential `(o, a, o_next, r, t)` 5-tuple, with a maximum queue size 1000000
    * The queue Stores sequentially the following
        * observations
        * actions
        * next observations
        * rewards
        * terminals 
    * Method: 
        * Takes several paths as input
        * Appends the above things sequentially to the queue
    * Method:
      Given a batch size (32) samples one batch of the above 

* Policy:
    * Has an MLP graph
    * Has an optimization OP
    * Method:
      * Given a batch of observations, return a batch of actions, using the MLP graph
    * Method:
      Given a batch of observations and a batch of actions, do one step gradient update of the policy, using the optimization OP

* Agent: 
    * Has a replay buffer
    * Has a (actor) policy
    * Method:
        * Given some episodes, update the replay buffer
    * Method:
        * Given a batch of 5-tuples, do one step update its policy
    * Method:
        * Samples a batch from the replaybuffer

# Training loop:




Critical hyperparameters:
* `n_iter`: number of Dagger iteration loops. Set to 100
* `batch_size`: how many **new** 5-tuples to collect for each iteration
* `train_batch_size`: training batch size
* `train_steps`: within each Dagger iteration, how many gradient updates to do.



The most important and difficult hyparameter is `batch_size`, how many new tuples to collect. Other things:

* `n_iter`: can be determined with tensorboard. the larger the better
* `train_batch_size`: as used in DL
* `train_steps`: the larger the better.

```python
policy = random_policy(lr=1e-3)
replay_buffer = empty_buffer(buffer_size=1000000)
agent = Agent(policy, replay_buffer)
# This refers to the number of new 5-tuples to collect
batch_size = B
# This refers to the actual batch size in training
train_batch_size = Btrain
# Training steps
train_steps = T
for i in range(n_iter):
  if i == 0:
		# A list of lists. Inner lists contains 5-tuples.
    # batch_size is not used here. the more the better
    paths = sample_trajectories(expert, expert, batch_size)
  else:
		# A list of lists. Inner lists contains 5-tuples. The total length will be largers than B
    paths = sample_trajectories(agent, expert, batch_size)
  # Concat all paths so we get a large list of 5-tuples. Add these tuples to replay buffer
  agent.add_replay_buffer(flatten_to_tuples(paths))
  for j in range(train_steps):
    # Sample a batch from replay buffer
    batch = agent.sample() 
    # One gradient step
    agent.train(batch)
```

So data:

* Data
  * initial: tuples from episodes of your expert
  * Else: collect a certain number of tuples with the policy
* add to buffer
* train using the **buffer**.


The most important thing seems to be `n_iter`, `train_steps`, and `batch_size`







Control:

* Number of data: fixed, 2000
* Batch size: 100
* Number of iterations: 1000
* Network depth: 2
* Width: 64
* Eval batch size: the total length of episode used for evaluation
* Tasks
  * Ant
  * Hopper
  * Walker
  * HalfCheetah
  * Humanoid
