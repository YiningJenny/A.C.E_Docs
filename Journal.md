# Week 7
- Test and combine all assets in Unity(2D sprites and 3D models)
- Implementing scene switching and animation interactions based on game flowcharts(from Zhou).
## 26th May
- Test gyro sensor with Zhou and Xiao, connect my laptop to projector
- Test conductive rubber tube(resistance) with Lieven and Zhou
## 27th May 
Current game flow recording: (https://github.com/YiningJenny/A.C.E_Docs/blob/main/gameRecord-27May.mp4)


![image](https://github.com/YiningJenny/A.C.E_Docs/assets/119497753/71fc7331-f1cd-4fa6-abdf-3cc52016aa80)
Our game starts with a random text that appears at the top of the screen accompanied by a sound hint, and the player has to choose the correct character based on the hint. Inspired by James' audio lecture, I think we can also add some background sound effects to make a more immersive environment, such as the sound of water running, rain, etc., depending on the meaning of the text.
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
