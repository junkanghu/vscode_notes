# Codes Usage
## pytorch


## numpy


## general
* get the shape of dimension 0 whatever it is tensor or ndarray
  ``` python
  len(a)  # a: [batch, m, n]
  ```
* get options
  1. use **configargparse**: generate .txt files. (source: NeRF-run_nerf.py)
     file format:
     ``` txt
     a = 100
     b = 1000 
     ``` 
     * usage
     ``` python
     import configargparse
     parser = configargparse.ArgumentParser()
     parser.add_argument('--config', is_config_file=True,help='config file path')

     parser = config_parser()
     args = parser.parse_args()
     ```
     ``` python
     python run_nerf.py --config path(./../lego.txt)
     ```
     * advantage: can edit options in shell by calling "--" or "-"
     * limitation: all the options should be defined at argparse first. 
  2. use **pyhocon.ConfigFactory**: generate .conf files. (source: invrender)
     file format: 
     ``` conf
     train{
        exp = ''
        n = 100
        out = [1, 2, 3, 4]
     }

     test{
        exp = ''
        key{
           alpha = 1.455
        }
     }
     ```
     * usage: 
       ``` python
       from pyhocon import ConfigFactory
       args = ConfigFactory.parse_file(path)
       n = args.get_int('train.n')
       exp = args.get_string('train.exp')
       alpha = args.get_float('test.key.alpha')
       out = args.get_list('train.out')
       ```
     * limitation: Everytime need the *get_{int or float}*
     * 


*  