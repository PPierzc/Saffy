# Sappy
Signal Analysis &amp; Processing in Python

A simple signal analysis tool, which aims at clarity of code and reproducibility of solutions. The plugin architecture
aims to encourage building modular code among scientists and data analysts. It provides a basic structure for signal
storage and a pipeline for analysis.

## Usage
```python
import Sappy
sig = Sappy.SignalManager(generator=signal_data)
```

#### Basic SignalManager instance structure
| field | description |
|--------|------|
| fs   | sampling frequency  |
| num_channels | number of channels |
| channel_names | name for each channel |
| data | the signal in the structure of (epoch x channel x signal) |
| t | time vector |
| epochs | number of epochs |
| tags | position of tags in signal |

#### SignalManager init function
It takes one labelled argument: `generator` or `filename`.

###### Generator
A dictionary of the structure
```
data = {
      'fs': # float,
      'num_channels': # integer,
      'channel_names': # list of strings,
      'epochs': # integer,
      't': # time array,
      'tags': # list,
      'data': # Signal Matrix
  }
```


## Plugins
Plugins are classes that inherit from the PluginManager. They extend the functionality of the basic Signal Manager.
Some plugins are provided out of the box

#### Filters
Adds basic filters

#### Graphics
Adds functions to display the signal data

#### Welch
Calculating the Welch Spectrum

#### Hilbert
Calculating the Hilbert Transform

### Creating Custom Plugins
You might want to add some custom features.

```python
import Sappy

class CustomPlugin(Sappy.PluginManager):
    def __init__(self):
        super().__init__()
        self.custom_param = 'some value'
        
    def custom_function(self):
        # do something
        pass
        
Sappy.SignalManager.register_plugin(CustomPlugin)

sig = Sappy.SignalManager(generator=signal_data)

sig.custom_function()
```

## Example
A short example of how to use Sappy for EEG data analysis.

```python
def generate_signal():
  data = {
      'fs': 512,
      'num_channels': 3,
      'channel_names': ['C3', 'C4', 'd1'],
      'epochs': 1
  }
  
  T = 20
  t = np.arange(0, T, 1 / data['fs'])
  
  mu_freq = 10
  beta_freq = 23
  net_freq = 50
  
  data['data'] = np.zeros((data['epochs'], data['num_channels'], len(t)))
  
  for epoch in range(data['epochs']):
    data['data'][epoch][0] += (0.1 * t + 0.1) * sin(t, mu_freq)
    data['data'][epoch][0] += (0.1 * t + 0.1) * sin(t, beta_freq)
    data['data'][epoch][0] += sin(t, net_freq)
    data['data'][epoch][0] += 0.3 * noise(t)

    data['data'][epoch][1] += (0.1 * t + 0.1) * sin(t, mu_freq)
    data['data'][epoch][1] += (0.1 * t + 0.1) * sin(t, beta_freq)
    data['data'][epoch][1] += sin(t, net_freq)
    data['data'][epoch][1] += 0.3 * noise(t)

    data['data'][epoch][2][::5*data['fs']] = 1
    data['data'][epoch][2][0] = 0
    
  data['t'] = t
  data['tags'] = []
  
  return data

EEG = Sappy.SignalManager(generator=generate_signal())

EEG.set_tags_from_channel('d1')
EEG.remove_channel('d1')

PRE_EEG = EEG.copy('pre')
PRE_EEG.set_epochs_from_tags(-4, -2)

PRE_EEG.welch_spectrum()
PRE_EEG.spectrum = np.mean(PRE_EEG.spectrum, axis=0)
PRE_EEG.spectrum = np.reshape(PRE_EEG.spectrum, (1, *PRE_EEG.spectrum.shape))

POST_EEG = EEG.copy('post')
POST_EEG.set_epochs_from_tags(0.5, 2.5)

POST_EEG.welch_spectrum()
POST_EEG.spectrum = np.mean(POST_EEG.spectrum, axis=0)
POST_EEG.spectrum = np.reshape(POST_EEG.spectrum, (1, *POST_EEG.spectrum.shape))

fig, ax = plt.subplots(
    nrows=max([PRE_EEG.num_channels, POST_EEG.num_channels]),
    ncols=1,
    sharex=True,
    sharey=True,
    figsize=(10, 10)
)

PRE_EEG.spectrum_plot(
    fig,
    ax,
    'Change',
    label='Pre'
)

POST_EEG.spectrum_plot(
    fig,
    ax,
    color='#0000ff',
    label='Post'
)

for a in ax:
  a.legend()

plt.show()
plt.close()
```
![alt text](examples/example.png)

## Contributing
If you like the project and want to add something to it then please create a pull request.
- The title should shortly summarize the goal of your addition
- In the description go in depth with the changes you have made and why.

