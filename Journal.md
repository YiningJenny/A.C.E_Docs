# Week 4
- Discuss and define the iterative prototype of the project, divide the work and establish a timetable. 
- Search for online resources to specify a project implementation plan.
## 3rd May （待补充）
- First group discussion, define the prototype of project.
- Timetable schedule and division of labour plan.
## 4th May
Tutorial with Julia, here are recourses and links that might be useful:
- Arduino sensor: conductive wire, (https://www.adafruit.com/product/519), (https://learn.adafruit.com/thermistor), (https://www.st.com/en/mems-and-sensors/lis3dh.html)
- Arduino sensor: string motor sensor,
- Unique texture in blender: https://www.youtube.com/watch?v=lAnUI-eTefE
- Blender how to export FBX with texture: https://www.youtube.com/watch?v=kEP34CbPWUo
- Import model with texture into Unity: (https://www.cgtrader.com/tutorials/1542-importing-an-fbx-file-into-unity-with-textures)
# Week 5
- Defining the art style， detailing project plan
## 10th May （待补充）
- 确定要建模的字
- 确定游戏流程
- 确定美术风格
# Week 6
- Determining the final output model for the project, identifying test and filming locations and purchasing materials
- Beginning of Arduino and Unity exploring.
## 19th May
- Order Arduino sensors and other materials that we need.

Conductive resistance link: (https://www.ebay.co.uk/itm/224068660324)
- Get Arduino code and gyro sensor from Zhou. Get all 3D word models from Zhou and Xiao.
- Since we need to import 3D models to a 2D Unity project, there is no light in the scene, we met rendering issue. 3D models look like 2D image

![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/a001d41b-13fc-4030-b002-07df51603c0b)

- To render 3D models, we need light. As the background was bright, the model was not rendered well, so I thought of adding a black translucent mask to the background.

![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/0b7f78c9-3b02-4763-8aee-1d66344859f8)

- I tried different light effects, but point light works best for now. I want to try combining different color of lights to match background better.

## 20th May



- Firstly I try to control the point light by mouse position. Here is how I code:
```C#
public Vector3 mousePosition;
void Update()
    {
        MouseFollow();
    }

void MouseFollow()
    {
        mousePosition = Camera.main.ScreenToWorldPoint(Input.mousePosition);
        mousePosition.z = -3f;
        transform.position = mousePosition;
    }
```
- Secondly I try to receive outputs from Arduino, and control the point light move. Here is the code: (in Unity)
```C#
SerialPort serialPort = new SerialPort("COM3", 115200);
float lightPosX;
float lightPosY;
void Start(){
  serialPort.Open();
}

void Update()
    {
        MouseFollow();
        CheckArduino();
    }

public void CheckArduino(){
    string data = serialPort.ReadLine();// ReadByte();
    if (data != ""){
        if (data.StartsWith("X:"))
        {
            string[] splitData = data.Split(':');
            float xValue = float.Parse(splitData[1]);
            print(xValue);
            lightPosX = xValue;
        }else if(data.StartsWith("Y:")){
            string[] splitData = data.Split(':');
            float yValue = float.Parse(splitData[1]);
            print(yValue);
            lightPosY = yValue;
        }
    }
    transform.position = new Vector3(lightPosX, lightPosY, -2.5f);
}
```
- The movement is very slow, but the sensor is really controlling the point light.

Test video link: (https://youtu.be/hcRylLPXoEU)
# Week 7
- Test and combine all assets in Unity(2D sprites, 3D models and sound effect)
- Implementing scene switching and animation interactions based on game flowcharts(from Zhou).

## 24th May
- I got game flow chart and sound effect from Zhou

## 25th May
- I got background images from Amy.

## <a name="26May"></a>26th May
- Test gyro sensor with Zhou and Xiao, connect my laptop to projector
- Test conductive rubber tube(resistance) with Lieven and Zhou

Testing video: (https://youtu.be/PiEtQ_JJlOA)

![1b9a7f806c703df14e0a3e3522dcc93](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/3dd99f09-69fe-499d-a10e-5f412a866fc9)

- Firstly we attached the Gyro sensor to the bow and tested the game by connecting my laptop to the projector. It is clear from the video that the sensor can control the Unity point light movement, but it is very insensitive.

![9471701356730ad66093982096e3ced](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/29fa33f3-82fa-47d8-94b0-c68ffb556e1e)
- We tested the conductive resistance (we wanted to replace the bowstring with this conductive resistance to make a determination with the gyro sensor). We tested the change in resistance between the relaxed and strained state to confirm that it would work for our project. The Value changed slightly, but I think I can simply use them by calculating.
- I asked Lieven and Seamus about gyro sensor and fix the insensibility issue. I think the problem lies in how I receive and process the data transferred from Unity to Arduino.

**My previous code block in Arduino: (Here I've only shown the part that went wrong, not the whole code)**
```C++
  Serial.print("X:");
  Serial.println(event.orientation.x, 4);
  Serial.print("Y:");
  Serial.println(event.orientation.y, 4);
  Serial.print("Z:");
  Serial.println(event.orientation.z, 4);
```

**My previous code block in Unity:**
```C#
SerialPort serialPort = new SerialPort("COM3", 115200);
string data = serialPort.ReadLine();// ReadByte();
if (data != ""){
    if (data.StartsWith("X:"))
    {
        string[] splitData = data.Split(':');
        float xValue = float.Parse(splitData[1]);
        print(xValue);
        lightPosX = xValue;
    }else if(data.StartsWith("Y:")){
        string[] splitData = data.Split(':');
        float yValue = float.Parse(splitData[1]);
        print(yValue);
        lightPosY = yValue;        
    }
}
```

This code structure means that this function is called once per frame, each time gets either the X value or the Y value, but it does not get X and Y in regular sequence; instead, it is completely random. That's why the output values in Arduino was quite fluent and smooth but not in Unity.

**My Unity code afterwards:**
```C#
SerialPort sp = new SerialPort("COM3", 115200);
Vector3 lightPos;
public void CheckArduino()
    {
        string data = sp.ReadLine();// ReadByte();
        string[] splitData1 = data.Split(',');
        float xValue = float.Parse(splitData1[0]);
        float yValue = float.Parse(splitData1[1]);
        float zValue = float.Parse(splitData1[2]);
        lightPos = new Vector3(-1*((xValue/360)*100-50), (yValue+45)/2+10, -2.5f);
        print(lightPos);
    }
```

- By this way I can receive a vector3 coordinate from Arduino per frame and use it directly, much more efficiency.

![53a86f2e74c8570977789344289b028](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/d0e29a14-a29e-4d05-bf9e-270f7039cf7a)

- After solving the value transmission problem we discussed how the gyro sensor works. The value it outputs from Arduino is the absolute rotation angle and in order to convert the angle value into length coordinates I need to normalise it, which I have calculated as shown in the diagram and code above. I don't think I've followed the most correct formula for it, I'm terrible at maths. But I just kept it since it seems to work for the project so far. 
- I forget to take a test video for this successful version, but I took a picture of how we attached gyro sensor to bow (show as below). The sensor must be in this position, in this direction, otherwise it will be shown in the game as moving in the opposite direction.

![d574d4bd558e7c993a3ce09eb7f7873](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/18592642-2319-4817-a12b-3285350513de)


## 27th May 
- Almost finish all in Unity!

Current game flow recording: (https://github.com/YiningJenny/A.C.E_Docs/blob/main/gameRecord-27May.mp4)

In case the github link doesn't work: (https://youtu.be/yf2vd75wkXg)


![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/71fc7331-f1cd-4fa6-abdf-3cc52016aa80)
Our game starts with a random text that appears at the top of the screen accompanied by a sound hint, and the player has to choose the correct character based on the hint. Actually we didn't realize that sound effects can influent and enhance game experience that much. But inspired by James' audio lecture, I think we can also add some background sound effects to make a more immersive environment, such as the sound of water running, rain, etc., depending on the meaning of the text. It also helps player understand and remember the chinese character.

![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/37d86c8d-11c7-4e11-a0db-5a1ebf8eb411)
![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/1b6e0b57-e467-440a-aab5-0888645e5158)
There are different animations for incorrect and correct choices, and if player chooses correctly, it leads to the next level. By the way, all these interactions will be done by sensors in the future, but for now I just use mouse clicks instead.

![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/540920cb-5b6b-4a75-b745-d7d915ac2162)
The second level is an animation of the evolution of Chinese characters from ancient to modern times, here in reverse order. I will replace it after group member has updated the animation assets. Again, hit on the text with the mouse instead of sensor, and the animation prompt appears if player succeed. In case of doubt, this level is intended to be understood and remembered by players (who don't know the Chinese character).
### Unity modules and code
- Moving 3D words

I create animation for each word model and record the movement by add key frame.
- Ramdomly generate 2D word

First I create a new list to hold all the words, then I use the indexed list to get the int value of each word
```C#
public List<GameObject> words = new List<GameObject>();
public int wordIndex;
Vector3 spawnPosition;

public void SpawnWord()
    {
        spawnPosition = new Vector3(0f, 3.6f, -1f);
        int index = Random.Range(0, words.Count);
        Instantiate(words[index], spawnPosition, Quaternion.identity);
        print("Index:" + index);
        wordIndex = index;
    }
```
Then check mouse click and target each word by position. Here I made a seemingly redundant step where I called the mouse position from the LightController class. That's because mouse click and mouse position will be replaced by sensor trigger in the future, I need the sensor value from LightController. I use Invoke() here for animation playing. In this function I also activate "great" or "try again" animation.
```C#
public LightController lightController;
if (wordIndex == 0 && Input.GetMouseButtonDown(0))
        {
            if (4.7f < lightController.mousePosition.x && lightController.mousePosition.x < 8.64f && lightController.mousePosition.y < 6.37f && lightController.mousePosition.y > 2.75f)
            {
                print("you are right");
                successDialog.SetActive(true);
                Invoke("CloseSuccessDialog", 2f);//wait for 3s and call CloseSuccessDialog
                Invoke("NextLevel", 3f);
            }
            else
            {
                tryAgainDialog.SetActive(true);
                Invoke("CloseSuccessDialog", 2f);
            }
        }
```
This is how I deactivate animation and switch game scene:
```C#
void CloseSuccessDialog()
    {
        successDialog.SetActive(false);
        tryAgainDialog.SetActive(false);
    }
void NextLevel()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex+1);
    }
```
_Amy通过AI软件调整了背景图像，但是忘记删除水印。所以我们这周开会之后调整为无水印的背景图像。_

# Week 8
- Finish sodering, link arduino and unity together, test and optimize code.
## 30th May  combine arduino code together

![c4c5c2d0131318ec7afe96829637521](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/dbaf6a47-f7f6-49b6-af51-5cabfb02b382)

Zhou finished three sensors' test seperatly, and combine the sensor to a big breadboard. I check and combine all the code together, also I remove some useless part and format the print out. Here is the _final version_ Arduino code for all sensors:

```C++
#include <Wire.h>
#include "Adafruit_MPR121.h"
#include <Adafruit_Sensor.h>
#include <Adafruit_BNO055.h>
#include <utility/imumaths.h>

// conductive wire
#ifndef _BV
#define _BV(bit) (1 << (bit)) 
#endif

// gyro 
#define BNO055_SAMPLERATE_DELAY_MS (100)
Adafruit_BNO055 bno = Adafruit_BNO055(55, 0x28, &Wire);

// pressure
#define FORCE_SENSOR_PIN A0 // the FSR and 10K pulldown are connected to A0

Adafruit_MPR121 cap = Adafruit_MPR121();
// conductive wire
uint16_t lasttouched = 0;
uint16_t currtouched = 0;

void setup() {
  Serial.begin(115200);
  
  //conductive wire
  // If tied to SDA its 0x5C and if SCL then 0x5D
  if (!cap.begin(0x5A)) {
    Serial.println("MPR121 not found, check wiring?");
    while (1);
  }
  Serial.println("MPR121 found!");

  // gyro
  if(!bno.begin())
  {
    Serial.print("Ooops, no BNO055 detected ... Check your wiring or I2C ADDR!");
    while(1);
  }
  bno.setExtCrystalUse(true);
}

void loop() {
  // pressure
  int analogReading = analogRead(FORCE_SENSOR_PIN);

  // gyro
    sensors_event_t event;
    bno.getEvent(&event);
    Serial.print("Vector3,");
    Serial.print(event.orientation.x, 4);
    Serial.print(",");
    Serial.print(event.orientation.y, 4);
    Serial.print(",");
    Serial.println(event.orientation.z, 4);
    // pressure
    Serial.print("Pressure:");
    Serial.println(analogReading);
    delay(50);
    
  // conductive wire
  currtouched = cap.touched();
  
  for (uint8_t i=0; i<12; i++) {
    // it if *is* touched and *wasnt* touched before, alert!
    if ((currtouched & _BV(i)) && !(lasttouched & _BV(i)) ) {
      //Serial.print(i); 
      Serial.println("Wire:1");//touched
    }
    // if it *was* touched and now *isnt*, alert!
    if (!(currtouched & _BV(i)) && (lasttouched & _BV(i)) ) {
      //Serial.print(i); 
      Serial.println("Wire:0");//released
    }
  }

  // reset our state
  lasttouched = currtouched;

  return; // important!! It works
    
  for (uint8_t i=0; i<12; i++) {
    Serial.print(cap.filteredData(i)); Serial.print("\t");
  }
  
  for (uint8_t i=0; i<12; i++) {
    Serial.print(cap.baselineData(i)); Serial.print("\t");
  }
  delay(50);
} 
```

I realize that the optimized code from [26th May](#26May) doesn't work for now. I need to do some further optimization to let Unity know which sensor each output value comes from Arduino. I'll try it tomorrow after Zhou finish soldering.

## 31th May
- I get background music from Xiao, I apply it to unity.
- We try to control unity by arduino, but the conductive wire that we use is very inefficient in conducting electricity. We might need to change another sensor later. Test video: https://www.youtube.com/shorts/A8Jz8CVUf9Y
## 2nd-3rd Jun  test final_final_final version arduino with unity, reconstruct code structure and final prototype test
- We invite friends from class to play our game, and then I found a bug. The "try again" and "great" animation effects appears together, it is not what I expect. (details show as below)
- Test link: https://youtu.be/HIJAXM3FciU
- I send the bug to slack technical channel, and get answer from Tom. Firstly, it might because that the collection of if statements is so complicate that it is hard for Unity to read and it’s bound to cause bugs. Secondly, I was using ```if(){}``` instead of ```else if(){}```, which means it can run after the first one, and they have a similar structure. That's the most probably reason why the two effects appear together. 
- To solve this, I change the structure of the code, by putting pieces of code that occur in more than one place in a function. I create a new function to judge the light position, and return a boolean value, so I don't need to bring a quite long conditional into ```if``` statement. 
- The optimized code:

```C#
public void LightPosition()
    {
        // rain
        if (lightController.lightPos.x > -14.1f &&
            lightController.lightPos.x < -8.96f &&
            lightController.lightPos.y < -1.67f &&
            lightController.lightPos.y > -8.06f)
        {
            rainPos = true;
            print("rainPos: " + rainPos);
        }

        else { 
            rainPos = false;
            print("rainPos: " + rainPos);
        }

        // sheep
        if (lightController.lightPos.x > 2.18f &&
            lightController.lightPos.x < 6.58f &&
            lightController.lightPos.y < 2.33f &&
            lightController.lightPos.y > -1.93f)
        {
            sheepPos = true;
            print("sheepPos: " + sheepPos);
        }
        else { 
            sheepPos = false;
            print("sheepPos: " + sheepPos);
        }
    }
    
    
    // trigger animation and switch game scenes
    public void DetectMouseCllick() {
        
        //rain
        if (wordIndex == 0 && 
            lightController.currentTouch == false && 
            lightController.lastTouch == true)
        {
            // bow release sound effect
            bowRelease.SetActive(true);
            Invoke("CloseSoundEffect", 2f);
            // triggerAnimation，switch game scene
            if (rainPos == true)
            {
                print("you are right");
                successDialog.SetActive(true);
                Invoke("CloseDialog", 3f);//等待3秒后call CloseSuccessDialog
                Invoke("NextLevel", 3f);
            }
            else if(rainPos == false)
            {
                tryAgainDialog.SetActive(true);
                Invoke("CloseDialog", 2f);
            }
        }
        //sheep
        if (wordIndex == 1 &&
            lightController.currentTouch == false && 
            lightController.lastTouch == true)
        {
            // bow release sound effect
            bowRelease.SetActive(true);
            Invoke("CloseSoundEffect", 2f);
            // triggerAnimation，switch game scene
            if (sheepPos == true)
            {
                print("you are right");
                successDialog.SetActive(true);
                Invoke("CloseDialog", 3f);//等待3秒后call CloseSuccessDialog
                Invoke("NextTwoLevel", 3f);
            }
            else if(sheepPos == false)
            {
                tryAgainDialog.SetActive(true);
                Invoke("CloseDialog", 2f);
            }
        }
        //bow Pulling Sound Effect
        else if (lightController.currentTouch == true && lightController.lastTouch == false)
        {
            bowPull.SetActive(true);
            Invoke("CloseSoundEffect", 2f);
        }
        lightController.lastTouch = lightController.currentTouch;
    }
```

- The last one test before final physical construction. Video link: https://youtu.be/5qI0WFmb6ys
- By the game test, we find that we don't have a full and smooth game flow. It might influence players' game experience in the future. I need to add user interface, like pause menu, sound volume, quit game etc.
