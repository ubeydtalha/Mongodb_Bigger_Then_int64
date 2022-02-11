
# Usage
## Note

Note that you will not be able to perform numerical operations in your database with this method .

Maybe, as a better method, instead of converting to str, it can be made into 64-bit arrays and numeric operations can be provided.

## For Any Mongodb Driver

If you try to insert any integer which is bigger than 64 bit , The library you are using will probably throw an error.

![prof2](https://i.ibb.co/C8MG8dB/resim-2022-02-11-035339.png)

In this case, two ideas came to my mind, either I was going to convert the numbers into strings and add a text with the type in front of them, and after receiving it, it would be converted to its original type according to that text, or I would use the method I mentioned here.

I turned integer and float values ​​into an object (dict) and added its value, type and (optional) controller random variable.


This function recursively finds integer and float values ​​and converts them to objects.
```python
def int_to_str(self,data : Union[dict,list]):
    """
    Convert all int and float to str in a list or dict
    """

    try:
        data_ = {}
        list_ = []
        if isinstance(data,list):
            for i in range(len(data)):
                if isinstance(data[i],(int)):
                    if data[i] > 1 :
                        list_.append({"CUSTOM_NUMBER":"CUSTOM_NUMBER","value":str(data[i]), "type":"int"})
                    else: 
                        data_[key] = data[key]
                elif isinstance(data[i],(float)):
                    if data[i] > 1 :
                       list_.append({"CUSTOM_NUMBER":"CUSTOM_NUMBER","value":str(data[i]), "type":"float"})
                    else: 
                        data_[key] = data[key]


                elif isinstance(data[i],dict):
                    list_.append(self.int_to_str(data[i]))
                elif isinstance(data[i],list):
                    list_.append(self.int_to_str(data[i]))
                else:
                    list_.append(data[i])
            return list_
        elif isinstance(data,dict):
            for key,value in data.items():
                if isinstance(data[key],(int)) and is:
                    if value > 1:
                        data_[key] = {"CUSTOM_NUMBER":"CUSTOM_NUMBER","value":str(data[key]), "type":"int"}
                    else: 
                        data_[key] = data[key]
                elif isinstance(data[key],(float)):
                    if value > 1:
                        data_[key] = {"CUSTOM_NUMBER":"CUSTOM_NUMBER","value":str(data[key]), "type":"float"}
                    else: 
                        data_[key] = data[key]
                elif isinstance(value,dict):
                    data_[key] = self.int_to_str(value)
                elif isinstance(value,list):
                    data_[key] = self.int_to_str(value)
                else:
                    data_[key] = data[key]

            return data_
        else:
            return data
    except Exception as e:
        exception_printer(e)
```

Note : I wanted the values ​​to be greater than one here, but you may want them to be greater than the maximum int64 value (9223372036854775807).

It shouldn't be forget that bool type is subclasses of int so True or False values equel 1 or 0.
![StackOverFlow](https://stackoverflow.com/questions/37888620/comparing-boolean-and-int-using-isinstance)

when i do a save to database like this.

![save](https://i.ibb.co/VtGHNBF/resim-2022-02-11-040825.png)


After doing this process, when I wanted a data in the database, it had to be returned to me in its original format, so I wrote the following function.

```python
def str_to_int(self,data):
    """
    Convert all str to int in a list or dict
    """
    try:
        data_ = {}
        list_ = []
        if isinstance(data,list):
            for i in range(len(data)):

                if isinstance(data[i],dict):

                    if data[i].get("CUSTOM_NUMBER") == "CUSTOM_NUMBER":
                        if data[i].get("type") == "int":
                            try:
                                list_.append(int(data[i].get("value",0)))
                            except Exception:
                                list_.append(data[i])
                        elif data[i].get("type") == "float":
                            try:
                                list_.append(float(data[i].get("value",0)))
                            except Exception:
                                list_.append(data[i])

                        continue
                    list_.append(self.str_to_int(data[i]))
                elif isinstance(data[i],list):
                    list_.append(self.str_to_int(data[i]))
                else:
                    list_.append(data[i])
            return list_

        elif isinstance(data,dict):
            for key,value in data.items():

                if isinstance(value,dict):
                    if data[key].get("CUSTOM_NUMBER") == "CUSTOM_NUMBER":
                        if data[key].get("type") == "int" :
                            try:
                                data_[key] = int(data[key].get("value",0))
                            except Exception:
                                data_[key] = data[key].get("value",0)
                        elif data[key].get("type") == "float":
                            try:
                                data_[key] = float(data[key].get("value",0))
                            except Exception:
                                data_[key] = data[key].get("value",0)
                        continue
                    data_[key] = self.str_to_int(value)
                elif isinstance(value,list):
                    data_[key] = self.str_to_int(value)
                else:
                    data_[key] = value
            return data_
        else:
            return data
    except Exception as e:
        exception_printer(e)
```

Thus, you can easily use numbers larger than int64 by using these functions before and after sending your data.

Here, I would like you to pay attention to the following, while filtering the database, you now need to access the value in that variable. I leave a small example for this.

```python

async def find_by_id(self,id):
    try:
        data = await self.db.find_one({"_id.value": str(id)})
        data = self.str_to_int(data)
        return data
    except Exception as e :
        exception_printer(e)
```

While saving data, you can save as follows.

```python

async def insert(self, data):
    try:
        #I recommend you to check whether there is an id in the data beforehand, or whether the data is none.
        await self.db.insert_one(self.int_to_str(dict))
    except Exception as e :
        exception_printer(e)

```

I hope I could help.
