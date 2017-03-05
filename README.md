# gym-mupen64plus

This project is an [OpenAI Gym](https://github.com/openai/gym/) environment wrapper for the [Mupen64Plus](http://www.mupen64plus.org/) N64 emulator. The goal of this project is to provide a platform for reinforcement learning agents to be able to easily interface with N64 games using the OpenAI gym library. This should allow for easier adaptation of existing agents that have been built using the gym library to play Atari games, for example.

Currently, only MarioKart64 has been wrapped, but the core environment has been built to support any games. This top-level README will be used to describe the basic setup instructions, architecture of the environment, etc. Each game that gets wrapped will have its own README file within its respective subdirectory. This top-level README will link to each of the games' README file as well.

#### Thanks:
Many of the core concepts for this wrapper were borrowed/adapted directly from [@kevinhughes27](https://github.com/kevinhughes27)'s fantastic [TensorKart](https://github.com/kevinhughes27/TensorKart) project (self-driving MarioKart with TensorFlow). A huge thanks goes out to him for inspiring the foundation of this project.


## Setup

### Python Dependencies
*If you follow the installation steps below, these dependencies will be resolved automatically.*
* Python 2.7
* gym
* numpy
* pygame
* PyYAML
* termcolor
* wx

### Additional Dependencies
* Mupen64Plus
    ```bash
    #!/bin/bash
    sudo apt-get install mupen64plus
    ```

* mupen64plus-input-bot
    ```bash
    #!/bin/bash
    mkdir mupen64plus-src && cd "$_"
    git clone https://github.com/mupen64plus/mupen64plus-core
    git clone https://github.com/kevinhughes27/mupen64plus-input-bot
    cd mupen64plus-input-bot
    make all
    ```
    *Note the path of the resulting .so file*

* One or more N64 ROMs (see the [Games](#games) section below)

### Installation

In order to set up the dependencies, it is recommended that you use a [conda environment](http://conda.pydata.org/docs/using/envs.html) to minimize disruption to your system and to prevent version conflicts with libraries you may already have installed. (*Alternatively, you can just execute the first and last statements to install directly if you prefer*).

```bash
#!/bin/bash
cd gym-mupen64plus

# Create the conda environment with all the necessary requirements
conda env create -f environment.yml

# Activate the new environment
source activate gym-mupen64plus

# Install the gym-mupen64plus package in the new environment
pip install -e .
```

### Configuration

A configuration file ([`config.yml`](gym_mupen64plus/envs/config.yml)) has been provided for the core wrapper where the primary settings are stored. This configuration will likely vary on your system, so please take a look at the available settings and adjust as necessary.

Additionally, each game environment may specify configuration values which will be stored in a separate config file in the game's specific subdirectory (see each game's README for those details).


## Architecture

### `Mupen64PlusEnv`:

The core `Mupen64PlusEnv` class has been built to handle many of the details of the wrapping and execution of the Mupen64Plus emulator, as well as the implementation of the gym environment. In fact, it inherits from `gym.Env`. The class is abstract and each game environment inherits from it. The game environment subclass provides the ROM path to the base.

#### Initialization:
* starts the controller server using the port specified in the configuration
* starts the emulator process with the provided ROM path (this also uses values from the config file)
* sets up the observation and action spaces (see the [gym documentation](https://gym.openai.com/docs))
    * the observation space is 

#### Methods:
* `_step(action)` handles taking the supplied action, passing it to the controller server, and reading the new `observation`, `reward`, and `end_episode` values.

* `_observe()` grabs a screenshot of the emulator window and returns the pixel data as a numpy array.

* `_render()` currently doesn't do anything. Eventually the project will support xvfb and this method will be used to make the emulator visible, when specified.

* `_close()` shuts down the environment: stops the emulator, and stops the controller server.

* Abstract methods that each game environment must implement:
    * `_navigate_menu()` moves through the game menu from startup to the beginning of an episode.

    * `_get_reward()` determines the reward for each step.

    * `_evaluate_end_state()` determines whether or not the episode is over.

    * `_reset()` resets the environment to begin a new episode.

### `ControllerHTTPServer`:

When initialized, will start an HTTP Server listening on the specified port. The server will listen for `GET` requests, but will wait to respond until `send_controls()` is called. Each time `send_controls()` is called, it will block and wait for the `GET` request to be processed (up to a configured timeout). In other words, the emulator will end up waiting indefinitely for a controller action, essentially waiting for an agent to `step()`.

### `EmulatorMonitor`:

This class simply polls the emulator process to ensure it is still up and running. If not, it prints the emulator process's exit code. Eventually this will also cause the environment to shutdown since the heart of it just died.


## Example Agents

### Simple Test:
The simplest example to test if the environment is up-and-running:
```python
#!/bin/python
import gym, gym_mupen64plus

env = gym.make('Mario-Kart-Luigi-Raceway-v0')

for i in range(88):
    (obs, rew, end, info) = env.step([0, 0, 0, 0, 0]) # NOOP until green light

for i in range(100):
    (obs, rew, end, info) = env.step([0, 0, 1, 0, 0]) # Drive straight

raw_input("Press <enter> to exit... ")

env.close()
```

### AI Agent:
A more complete example AI agent will be linked later.


## Games

*Links to ROM files will not be included here. Use your ninja skills as appropriate.*

Here is a list of games that have been wrapped. Each game may support multiple 'modes' with different levels or missions configured. See each of the games' pages for more details.
* [MarioKart64](gym_mupen64plus/envs/MarioKart64/README.md)


## Contributing

Please create issues as you encounter them. Future work and ideas will be captured as issues as well, so if you have any ideas of things you'd like to see, please add them. Also, feel free to fork this repository and improve upon it. If you come up with something you'd like to see incorporated, submit a pull request. Adding support for additional games would be a great place to start.