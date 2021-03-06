# microprediction

Free access to live univariate time series prediction at www.microprediction.org where bots fight to predict you time series

    pip install microprediction 
    
This library can also be used to contribute predictions as well as soliciting them. 
    
    
# Quickstart 

See /polling_examples for stream creation examples, which look like this. 

    feed = MicroPoll(write_key=TRAFFIC_WRITE_KEY,
                     name='verrazano_speed.json',
                     func=verrazano_speed,
                     interval=20)
    feed.run() 
    
Once a stream is created you can view predictions at www.microprediction.org or by using the reader to get the cumulative distribution function. 
    
# Class Hierarchy 

    Conventions   
       |
    Reader
       |
    Writer ------------
       |               |
    Polling         Crawler
    
    
# Read client

It is possible to retrieve most quantities at www.microprediction.org with direct web calls such as https://www.microprediction.org/live/cop.json. For example:

    import requests
    lagged_values = requests.get('https://www.microprediction.org/live/lagged_values::cop.json').json()
    lagged        = requests.get('https://www.microprediction.org/lagged/cop.json').json()

However the reader client adds a little convenience. 

    from microprediction import MicroReader
    mr = MicroReader()
 
    summary       = mr.get_summary('cop.json')
    current_value = mr.get('cop.json')
    lagged_values = mr.get_lagged_values('cop.json') 
    lagged_times  = mr.get_lagged_times('cop.json')
    delayed       = mr.get_delayed('cop.json',delay=70)
    
Your best reference is the code https://github.com/microprediction/microprediction/blob/master/microprediction/reader.py 
    
# Write client

The write client is used to submit predictions or to create a data stream. 

## Submitting predictions 

To predict a data stream at www.microprediction.org is to supply a collection of scenarios. These scenarios are quarantined for different horizons (see delays parameter at https://www.microprediction.org/config.json ). When
the data is updated by the stream owner, rewards are calculated. People and machines making accurate probabilistic forecasts will see their balances (at www.microprediction.org/balance/YOUR_WRITE_KEY)
rise. 

### Step 1: Obtaining a write_key (muid.org)

Click on http://www.muid.org/create/ to create a write_key, though it is better to create a rarer one. Hash memorable keys are explained at https://vimeo.com/397352413   
    

### Step 2: Instantiate a writer 

    from microprediction import MicroWriter
    mw = MicroWriter(write_key=)    # Sub in your own write_key 
    
### Step 3: Submitting scenarios 
    
    scenarios = [ i*0.001 for i in range(mw.num_predictions) ] 
    mw.submit(name='cop.json',values=scenarios)    

There is no difference when predicting regular streams and derived streams. For example to predict the implied z-score: 

    my_scenarios = sorted(list(np.random.randn(mw.num_predictions))
    mw.submit(name="z1~airp-06820.json", write_key="ce169feeb3565b282d50a850dc62e0db", values = my_scenarios, delay=15)

### Step 4: Examine performance 

Visit leaderboards such as www.microprediction.org/leaderboards/cop.json or look across all streams with:

    mw.get_performance()

# Submitting data to be predicted

You can also use the writer to create a stream of live data that clever algorithms and humans can predict. 

    mw = MicroWriter(write_key=write_key)

However there is a higher barrier to entry...

### Step 1: Obtaining a rare write_key

To create a new stream you need:

    muid.difficulty(write_key)  >  official minimum difficulty     # 12 at time of writing

See https://www.microprediction.org/config.json for the current values of min_len, which is the official minimum difficulty to create a stream. 
To mine for write_keys with this property you can cut and paste this bash command into terminal:

    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/microprediction/muid/master/examples/mine_from_venv.sh)"

or use the MUID library (www.muid.org) ...
    
    $pip install muid
    $python3
    >>> import muid
    >>> muid.mine(skip_intro=True)
    
See www.muid.org or https://vimeo.com/397352413 for more on MUIDs. Use a URL like http://www.muid.org/validate/fb74baf628d43892020d803614f91f29 to 
reveal the hidden "spirit animal" in a MUID. The difficulty is the length of the animal, not including the space.     

### Step 2: Updating the current value 

To create a new live data source or update its value:

    prctl = mw.put(name='mystream.json',value=3.14157) 

By default this returns a percentile so you know how surprising the data point is, relative to the CDF of predictions
made by others at some time in the past.   

### Step 3: Retrieve the distribution of future values

You can see what others think about the future of your data as follows:

     cdf = mw.get_cdf('cop.json',delay=mr.delays[0],values=[0,0.5])
     
where the delay parameter, in seconds, acts as a prediction horizon. This call will reveal the probability that your future value is less than 0.0, and the probability that it is 
less than 0.5. 
         
### Step 4: Hope that your write_key does not go broke ... or do something about it

When you create a stream you automatically participate in the prediction of the stream. A benchmark empirical sampling algorithm with some recency adjustment is used for this
purpose. If nobody can do a better job that this, your write_key balance will generally neither rise nor fall.  

However once smart people and algorithms enter the fray, you can expect this default model to be beaten and the balance on your write_key to trend downwards. 
On an ongoing basis you also need the write_key balance not to fall below a threshold bankruptcy level. The minimum balance for a key of difficulty 9 is also found at https://www.microprediction.org/config.json and the formula
 -1.0*( abs(self.min_balance)*(16**(write_key_len-9)) ) supercedes whatever is written here. However at time of writing the bankruptcy levels are:

|  write_key difficulty   |  bankruptcy         |  write_key difficulty   |  bankruptcy         |
|-------------------------|---------------------|-------------------------|---------------------|
|  8                      |  -0.01              |     11                  |   -256              |
|  9                      |  -1.0               |     12                  |   -4,096            |
| 10                      |  -16.0              |     13                  |   -65,536           |
       
Balance may be transfered from one write_key to another if the recipient write_key has a negative balance. You can use the transfer function to keep
a write_key alive that you need for sponsoring a stream. You can also ask others to mine muids for you and contribute in this fashion. However you cannot use a transfer to 
raise the balance associated with a write_key above zero. It is only possible to do that by means of accurate prediction. 

# Polling live data

The class MicroPoll provides turnkey creation of a datastream. See polling_examples. For instance to predict traffic speed every 
twenty minutes:

    feed = MicroPoll(write_key=TRAFFIC_WRITE_KEY,
                     name='verrazano_speed.json',
                     func=verrazano_speed,
                     interval=20)
    feed.run() 


To avoid the sponsoring write_key for a stream going bankrupt, the MicroPoll class performs some periodic mining and when it gets, lucky, makes
a transfer. 

# Crawling

To predict many streams you can derive from MicroCrawler. At minimum you'll want to override the sample() method that makes predictions based on lagged values. For example:


    class MyCrawler(MicroCrawler):

        def sample(self, lagged_values, lagged_times=None, **ignored ):
            """ Fat tails """
            return [1.02*s*(1+0.1*abs(s)) for s in sorted(np.random.randn(self.num_predictions)) ]  
            
Then 

    crawler = MyCrawler(write_key=MY_KEY)
    crawler.run()

# Higher dimensional prediction (copulas, Z-curves)

Advanced functionality is available to those with write_keys of difficulty 1 more than the stream minimum (i.e. 12+1). 
         
         
# Troubleshooting 
         
A few gotchas:
         
### Stream name rules 

 - Must end in .json  
 - Must contain only alphanumeric, hyphens, underscores, colons (discouraged) and at most one period.
 - Must not contain double colon. 

We keep open the possiblility of incorporating other data formats in the future, such as Arrow, but for now everything is JSON. 
         
### Logging 

Try:
    
    mw.get_errors()
    mw.get_warnings()
    me.get_confirmations()
    
which are also available directly. For example:

    error_log = requests.get('https://www.microprediction.org/live/errors::53e6fbba-2dcd-486c-a4ab-14759db58dde.json').json()
    error_log = requests.get('https://www.microprediction.org/errors/53e6fbba-2dcd-486c-a4ab-14759db58dde').json()
    
 
