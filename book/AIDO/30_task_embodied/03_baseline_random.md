# Agent template for `aido1_lf1` {#challenge-aido1_lf1-template-random status=ready}

This section describes the contents of the simplest baseline: a "random" agent.

## Repository

Check out [the repository](https://github.com/duckietown/challenge-aido1_LF1-template-random):

    $ git clone -b v3 git@github.com:duckietown/challenge-aido1_LF1-template-random.git

## Submit

Change into the directory:

    $ cd challenge-aido1_LF1-template-random
    
First, let's try if your environment is configured correctly, by checking that 
you are able to submit using the command:

    $ dts challenges submit 

If it works, you will be able to see your submission [here](https://challenges.duckietown.org/v3/humans/challenges/aido1_LF1-v3). 

But you will probably not appear in the [leaderboard](https://challenges.duckietown.org/v3/humans/challenges/aido1_LF1-v3/leaderboard).


## Anatomy of the submission

The submission consists of the following files:

    submission.yaml
    Dockerfile	
    requirements.txt	
    solution.py	

### `submission.yaml`
    
The file `submission.yaml` contains the configuration for this submission:

```yaml
challenge: aido1_LF1-v3     # which challenge to submit to
protocol: aido1_remote1-v3  # which protocol this submission can speak
user-label: "Random execution"
```
    
You may change the `user-label` to any short string that you want

### `requirements.txt`

This file contains the requirements that are need to run the "glue" code.

The requirements for interfacing with the server are:

* `duckietown-shell`
* `duckietown-challenges`

The requirements for connecting to the simulator are:

* `gym-duckietown-agent`
* `duckietown-slimremote`


Feel free to *add* but not subtract to this file.


### `solution.py` boilerplate

All the submissions use the same boilerplate code:

```python
class Submission(ChallengeSolution):
    def run(self, cis):
        assert isinstance(cis, ChallengeInterfaceSolution) 
        
        # run the actual code here
        my_solution()
        
        cis.set_solution_output_dict({})
        

if __name__ == '__main__':
    print('Starting submission')
    wrap_solution(Submission())
```

This boilerplate is used to make sure that the evaluation and server containers are properly synchronized.
More details about this are [here](#api-solution).

### `solution.py` agent connection code

Moving on to the `solve()` function, we see it has the following pattern:

```python
def solve(gym_environment, cis):
    
    # the Gym environment knows how to connect to the container
    env = gym.make(gym_environment)
    
    # typical Gym loop
    
    observation = env.reset()
    # While there are no signal of completion (simulation done)
    # we run the predictions for a number of episodes, don't worry, we have the control on this part
    while True:
        action = YOUR_AGENT(observation)
        # we tell the environment to perform this action and we get some info back in OpenAI Gym style
        observation, reward, done, info = env.step(action)
        
        # it is important to check for this flag, the Evalution Engine will let us know when should we finish
        # if we are not careful with this the Evaluation Engine will kill our container and we will get no score
        # from this submission
        if 'simulation_done' in info:
            break
            
        # end of an episode
        if done:
            env.reset()
```

Please follow the following rules:

* When you get `done = False`, the episode is over. Call `env.reset()`.
* When you get `simulation_done` in the `info` dict, break and exit cleanly.

The `action = YOUR_AGENT(observation)` line is where you should put the computation of the action.

Read on to know more details about who to integrate this template with tensorflow.
