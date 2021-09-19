# TelebotForms
Small extension for [pyTelegramBotAPI](https://github.com/eternnoir/pyTelegramBotAPI) library for building interactive forms in Telegram bots.

## Install
```
$ pip install TbfForms
```
## Example build simple form
```python
from tb_forms import TelebotForms,BaseForm,fields
from telebot import TeleBot

bot = TeleBot("your_token")
tbf = TelebotForms(bot)

class TestRegisterForm(BaseForm):
    update_name = "submit_register_form"
    form_title = "TBF Test Register Form"
    name = fields.StrField("Name","Enter your name:")
    age = fields.NumberField("Age","Select your age:",only_int=True,key_mode=True)
    sex = fields.ChooseField("Sex","Select your sex:",answer_list=["male","female"])
    photo = fields.MediaField("Photo","Enter your photo:",valid_types=['photo'],required=False,error_message="Error. You can only send a photo")
    freeze_mode = True
    close_form_but = False
    submit_button_text = "Register"

@bot.message_handler(commands=['start'])
def start_update(message):
    tbf.send_form(message.chat.id,TestRegisterForm())

@tbf.form_submit_event("submit_register_form")
def submit_register_update(call,form_data):
    print(form_data) # Completed form data
    bot.send_message(call.message.chat.id,"Successful registration")

```
## Docs

### TelebotForms

#### Init

| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| bot  | Yes  |   | telebot.TeleBot object  |pyTelegramBotAPI bot object |
| fsm  | No  | tb_forms.ffsm.MemoryFSM | tb_forms.ffsm.FSM | TbfForms FSM object |

#### Settings
```python
tbf = TelebotForms(bot)

# Global icon for a missing value field
tbf.GLOBAL_MISSING_VALUE_ICON: str = "💢"

# Global icon for a field with data  
tbf.GLOBAL_EDIT_ICON: str = "✏️"

''' Global cancel form button text
  Type: Union[Str,Callable]   '''
tbf.GLOBAL_CANCEL_BUTTON_TEXT = "Cancel"
# Callable example
tbf.GLOBAL_CANCEL_BUTTON_TEXT = lambda user_id: "Cancel"



# Global submit form button text
tbf.GLOBAL_SUBMIT_BUTTON_TEXT: str = "Submit"

# Global close button in form
tbf.GLOBAL_CLOSE_FORM_BUT: bool = True

''' Global form freeze mode.
If True, prohibits any action prior to submitting or canceling the  form. '''
tbf.GLOBAL_FREEZE_MODE: bool = True

''' Global freeze mode alart text.
If freeze_mode is True, sends this text after any action.
  Type: Union[Str,Callable]   '''
tbf.GLOBAL_STOP_FREEZE_TEXT = "Cancel or submit the form first before proceeding further"

''' Global invalide input error text.  
  Type: Union[Str,Callable]   '''
tbf.GLOBAL_INVALID_INPUT_TEXT = "Error. Invalide input!"

```

#### Methods

##### Send forms
* send_form - Send form to chat

| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| chat_id  | Yes  |   | int  |Chat id for send form |
| form  | Yes  |  | tb_forms.BaseForm | TbfForms Form object |

##### Handle form
* form_submit_event - handle submit form event

| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| update_name | Yes  |   | str  |update_name of form to handle |
```python
@tbf.form_submit_event("update_name")
def submit_update(call,form_data):
    pass
```
* form_cancel_event - handle cancel form event

| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| update_name | Yes  |   | str  |update_name of form to handle |
```python
@tbf.form_cancel_event("update_name")
def cancel_form_update(call,form_data):
    pass

```
* form_event - handle all form event

| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| update_name | Yes  |   | str  |update_name of form to handle |
| action | Yes  |   | list  | handle events type |
```python
@tbf.form_event("update_name",action=["submit","cancel"])
def form_event_update(call,form_data):
    print(form_data.update_action) # event action type
```
### BaseForm
#### Parameters
| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| update_name | Yes  |   | str  |update name for handle event |
| MISSING_VALUE_ICON | No  | "💢"  | str  | Icon for a missing value field |
| EDIT_ICON | No  | "✏️"  | str  | Icon for a field with data  |
| freeze_mode | No  | False  | bool  | Form freeze mode. If True, prohibits any action prior to submitting or canceling the  form. |
| close_form_but | No  | False  | bool  | Show close form button  |
| submit_button_text | No  | "Submit"  | str  | Form submit button text  |
| cancel_button_text | No  | "Cancel"  | str  | Cancel form\input button text  |
| input_not_valid | No  | "Invalid input..."  | str  | Invalide input default error message text  |
| form_global_error_message | No  |   | str  | Global form default error message text  |
| form_valid_error | No  | "Error! You may have filled in some of the fields incorrectly. ⚠️" | str  | Form default pre-submit validation error message text |

#### Fields
```python
from tb_forms import fields
```

##### StrField 
Simple input text
```
-> str
```
| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| title | Yes  |   | str  |Field button title in form |
| input_text | Yes  |  | str  | Input message text |
| required | No  | True  | bool  | required for submit? |
| default_value | No  |  | str  | Default field value |
| validators | No  |  | List[Callable \| tb_forms.validators.Validator  ]  | Default field value |
| error_message | No  |  | str  | Validation error message text |

##### MediaField
Telegram Media input field
```python
-> tb_forms.tbf_types.MediaData
```
| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| title | Yes  |   | str  |Field button title in form |
| input_text | Yes  |  | str  | Input message text |
| required | No  | True  | bool  | required for submit? |
| validators | No  |  | List[Callable \| tb_forms.validators.Validator  ]  | Default field value |
| error_message | No  |  | str  | Validation error message text |
| valid_types | No  | All  | List[str]  | Aviable content_type for input |
| caption_required | No  | False | bool  | Required caption with media |
| only_text_aviable | No  | False | bool  | Aviable only text message |

##### NumberField
Input int or float value
```python
-> Union[int,float]
```
| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| title | Yes  |   | str  |Field button title in form |
| input_text | Yes  |  | str  | Input message text |
| required | No  | True  | bool  | required for submit? |
| validators | No  |  | List[Callable \| tb_forms.validators.Validator  ]  | Default field value |
| error_message | No  |  | str  | Validation error message text |
| only_text_aviable | No  | False | bool  | Input only int value |
| key_mode | No  | False | bool  | Input value from inline keyboard |
| input_range | No  | (1,99) | tuple  | key_mode input range |

##### BooleanField
True\False input
```python
-> bool
```
| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| title | Yes  |   | str  |Field button title in form |
| input_text | Yes  |  | str  | Input message text |
| required | No  | True  | bool  | required for submit? |
| default_value | No  |  | str  | Default field value |
| validators | No  |  | List[Callable \| tb_forms.validators.Validator  ]  | Default field value |
| error_message | No  |  | str  | Validation error message text |

##### ChooseField
Select input from list of values
```python
-> bool
```
| Args  | Required? |  Default  |  Type   | Description     |
| ------------- | ------------- |------------- |------------- |------------- |
| title | Yes  |   | str  |Field button title in form |
| input_text | Yes  |  | str  | Input message text |
| required | No  | True  | bool  | required for submit? |
| default_value | No  |  | str  | Default field value |
| validators | No  |  | List[Callable \| tb_forms.validators.Validator  ]  | Default field value |
| error_message | No  |  | str  | Validation error message text |
| answer_list | Yes  | [] | list  | Values for select |
| multiple | No  | False | bool  | Aviable multiple select |
| answer_mapping | No  |  | dict  | Dictionary for replace return selected value |

