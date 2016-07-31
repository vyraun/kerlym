# KEras Reinforcement Learning gYM agents, KeRLym

This repo is intended to host a handful of reinforcement learning agents implemented using the Keras (http://keras.io/) deep learning library for Theano and Tensorflow.
It is intended to make it easy to run, measure, and experiment with different learning configuration and underlying value function approximation networks while running a variery of OpenAI Gym environments (https://gym.openai.com/).

![Screenshot img](/examples/example.png?raw=true "KeRLym Screenshot")

# Agents

 - pg: policy gradient method with Keras NN policy network
 - dqn: q-learning agent with Keras NN Q-fn approximation (/w concurrent actor-learners)

# Installation

```
sudo python setup.py install
```

# Usage

```
./run_pong.sh
```

or

```
Exmaple: kerlym -e Go9x9-v0 -n simple_dnn -P

Usage: kerlym [options]

Options:
  -h, --help            show this help message and exit
  -e ENV, --env=ENV     Which GYM Environment to run [Pong-v0]
  -n NET, --net=NET     Which NN Architecture to use for Q-Function
                        approximation [simple_dnn]
  -b BS, --batch_size=BS
                        Batch size durring NN training [32]
  -o DROPOUT, --dropout=DROPOUT
                        Dropout rate in Q-Fn NN [0.5]
  -p EPSILON, --epsilon=EPSILON
                        Exploration(1.0) vs Exploitation(0.0) action
                        probability [0.1]
  -D EPSILON_DECAY, --epsilon_decay=EPSILON_DECAY
                        Rate of epsilon decay: epsilon*=(1-decay) [1e-06]
  -s EPSILON_MIN, --epsilon_min=EPSILON_MIN
                        Min epsilon value after decay [0.05]
  -d DISCOUNT, --discount=DISCOUNT
                        Discount rate for future reards [0.99]
  -t NFRAMES, --num_frames=NFRAMES
                        Number of Sequential observations/timesteps to store
                        in a single example [2]
  -m MAXMEM, --max_mem=MAXMEM
                        Max number of samples to remember [100000]
  -P, --plots           Plot learning statistics while running [False]
  -F PLOT_RATE, --plot_rate=PLOT_RATE
                        Plot update rate in episodes [10]
  -a AGENT, --agent=AGENT
                        Which learning algorithm to use [dqn]
  -i, --difference      Compute Difference Image for Training [False]
  -r LEARNING_RATE, --learning_rate=LEARNING_RATE
                        Learning Rate [0.0001]
  -E PREPROCESSOR, --preprocessor=PREPROCESSOR
                        Preprocessor [none]
  -R, --render          Render game progress [False]
  -c NTHREADS, --concurrency=NTHREADS
                        Number of Worker Threads [1]
```

or

```python
from gym import envs
env = envs.make("SpaceInvaders-v0")

import kerlym
agent = kerlym.agents.dqn.DQN(env, 
                    nframes=1, 
                    epsilon=0.5, 
                    discount=0.99, 
                    modelfactory=kerlym.networks.simple_cnn,
                    batch_size=32, 
                    dropout=0.1, 
                    enable_plots = True, 
                    epsilon_schedule=lambda episode,epsilon: max(0.1, epsilon*(1-1e-4)),                
                    dufference_obs = True,
                    preprocessor = kerlym.perproc.karpathy_preproc,
                    learning_rate = 1e-4
                    )
agent.learn()
```

# Custom Action-Value Function Network

```
def custom_Q_nn(agent, env, dropout=0, h0_width=8, h1_width=8, **args):
    S = Input(shape=[agent.input_dim])
    h = Reshape([agent.nframes, agent.input_dim/agent.nframes])(S)
    h = TimeDistributed(Dense(h0_width, activation='relu', init='he_normal'))(h)
    h = Dropout(dropout)(h)
    h = LSTM(h1_width, return_sequences=True)(h)
    h = Dropout(dropout)(h)
    h = LSTM(h1_width)(h)
    h = Dropout(dropout)(h)
    V = Dense(env.action_space.n, activation='linear',init='zero')(h)
    model = Model(S,V)
    model.compile(loss='mse', optimizer=RMSprop(lr=0.01) )
    return model

agent = keras.agents.D2QN(env, modelfactory=custom_Q_nn)

```

# Citation

If using this work in your research, citation of our publication introducing this platform would be greatly appreciated!
The arXiv paper is available at https://arxiv.org/abs/1605.09221 and a simple bibtex entry is provided below.

```
@misc{1605.09221,
Author = {Timothy J. O'Shea and T. Charles Clancy},
Title = {Deep Reinforcement Learning Radio Control and Signal Detection with KeRLym, a Gym RL Agent},
Year = {2016},
Eprint = {arXiv:1605.09221},
}
```

# Acknowledgements

Many thanks to the projects below for their inspiration and contributions
 - https://github.com/dandxy89/rf_helicopter
 - https://github.com/sherjilozair/dqn
 - https://gist.github.com/karpathy/a4166c7fe253700972fcbc77e4ea32c5
 - https://github.com/coreylynch/async-rl
 - Keras, Gym, TensorFlow and Theano

-Tim

# Remind
Install `pip install gym` and `pip install gym[atari]`. If gym[atari] has install error, `apt-get install cmake`.

# Error

I get: ./run_pong.sh

Traceback (most recent call last):
  File "/usr/local/bin/kerlym", line 4, in <module>
    __import__('pkg_resources').run_script('KeRLym==0.0.2', 'kerlym')
  File "/usr/local/lib/python2.7/dist-packages/pkg_resources/__init__.py", line 719, in run_script
    self.require(requires)[0].run_script(script_name, ns)
  File "/usr/local/lib/python2.7/dist-packages/pkg_resources/__init__.py", line 1517, in run_script
    exec(script_code, namespace, namespace)
  File "/usr/local/lib/python2.7/dist-packages/KeRLym-0.0.2-py2.7.egg/EGG-INFO/scripts/kerlym", line 6, in <module>
    
  File "build/bdist.linux-x86_64/egg/kerlym/__init__.py", line 1, in <module>
  File "build/bdist.linux-x86_64/egg/kerlym/agents.py", line 1, in <module>
  File "build/bdist.linux-x86_64/egg/kerlym/pg.py", line 4, in <module>
ImportError: No module named gym

