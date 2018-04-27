---
layout: post
title: "Custom shortcuts for common tasks in Unity"
excerpt: Custom shortcuts for common tasks in Unity
summary: This post teaches how to create a script to do common tasks in Unity editor easily
categories: [How To]
tags: [Unity, Unity 3D, Hierarchy, Customize, Customise, Editor, Window, C Sharp, gaming, menu, shortcut]
comments: true
---

One of the excellent plus points of Unity Editor is its extendability. You can easily extend its functionalities by writing some code to make working on the Unity Editor more convenient. Shared below are some of the useful codes I use to make working in Unity Editor faster and more efficient.

### Reset Transform
This is probably most of the people's top frequent task on Unity Editor. Whenever we place a GameObject on the scene, Unity tends to place it in a random position in the scene. So what people do is to click on the gear symbol on the Transform Section of the Inspector and select Reset to reset its transform for housekeeping or out of necessity. The code below will allow you to have it as a MenuItem to reset the GameObject selected and a shortcut key Alt+r to do this frequent task from you. This code comes adapted from [Github Gist](https://gist.github.com/thebeardphantom/ea6362139ee195a8abce){:target="_blank"} of thebeardphantom. In fact, some of my other useful codes use the same code to select the gameobjects. You can also do in bulk this task which makes it even more effecient.
{% highlight csharp%}
[MenuItem("GameObject/Reset Transform &r")]
static void ResetTransform()
{
    GameObject[] selection = Selection.gameObjects;
    if (selection.Length < 1) return;
    Undo.RegisterCompleteObjectUndo(selection, "Zero Position");
    foreach (GameObject go in selection)
    {
        InternalZeroPosition(go);
        InternalZeroRotation(go);
        InternalZeroScale(go);
    }
}
private static void InternalZeroPosition(GameObject go)
{
    go.transform.localPosition = Vector3.zero;
}
private static void InternalZeroRotation(GameObject go)
{
    go.transform.localRotation = Quaternion.Euler(Vector3.zero);
}
private static void InternalZeroScale(GameObject go)
{
    go.transform.localScale = Vector3.one;
}
{% endhighlight %}
### Reset Name
This comes from the annoyance of an updated unity feature(I think it is added after version 4.6) where if you try to add another gameobject(e.g., via duplicate, create or drag and drop) to the hierarchy or scene. If there is already a gameobject with the same name, it will add a suffix of ascending number enclosed with parentheses. See below to know what I mean and how ugly and untidy it looks. (it wasn't that bad last time...sobz...sobz)
<img src="{{ site.baseurl }}/images/unity_Custom_Shortcuts_1.png" alt="">
The code below will allow you to have it as a MenuItem to rename the GameObject removing the suffix or a shortcut key Alt+n to do this satisfying task for you. Better still, you can select in bulk to rename at one go!
{% highlight csharp%}
[MenuItem("GameObject/Reset Name &n")]
static void ResetName()
{
    GameObject[] selection = Selection.gameObjects;
    if (selection.Length < 1) return;
    Undo.RegisterCompleteObjectUndo(selection, "Reset Name");
    foreach (GameObject go in selection)
    {
        Rename(go);
    }
}
private static void Rename(GameObject go)
{
    int start = go.name.IndexOf("(");
    print(start);
    int end = go.name.IndexOf(")");
    print(end);
    if (start != -1 && end != -1 && start < end)
    {
        print("rename");
        go.name = go.name.Substring(0, start);
    }
}
{% endhighlight %}
### Lock and Unlock Inspector
Another frequently performed task. You will need to lock the inspector a lot of times when trying to drag objects to the inspector as it happens quite frequently the focus in inspector will accidentally move to the objects you want to drag instead. The code is actually from an [old unity forum thread](https://forum.unity.com/threads/shortcut-key-for-lock-inspector.95815/){:target="_blank"}. The code below will allow you to toggle lock and unlock on the <keyword>active inspector only</keyword> via a MenuItem or a shortcut key Alt+l to do this common task for you. 
{% highlight csharp%}
[MenuItem("GameObject/Toggle Lock Inspector &l")]
static void LockInspector()
{
    ActiveEditorTracker.sharedTracker.isLocked = !ActiveEditorTracker.sharedTracker.isLocked;
    ActiveEditorTracker.sharedTracker.activeEditors[0].Repaint();
}
{% endhighlight %}
### Revert to prefab
There are times where you want to revert a prefab instance changes to the same as its prefab. This useful code will help you do that via a MenuItem or shortcut key Alt+p. And you can do this in bulk!
{% highlight csharp%}
[MenuItem("GameObject/Revert Prefab &p")]
static void RevertPrefab()
{
    GameObject[] selection = Selection.gameObjects;
    if (selection.Length < 1) return;
    Undo.RegisterCompleteObjectUndo(selection, "Revert Prefab");
    foreach (GameObject go in selection)
    {
        if (PrefabUtility.GetPrefabType(go) == PrefabType.PrefabInstance)
        {
            PrefabUtility.RevertPrefabInstance(go);
        }
    }
}
{% endhighlight %}
### Apply changes to Prefab
This is to apply the changes done to a prefab instance to a prefab. You can do this via a MenuItem or shortcut key Alt+a.
<div class="warning">Caution: Doing this in bulk selection will result in the changes in the last item selected to overwrite other prefab instances if they are chosen as well</div>
{% highlight csharp%}
[MenuItem("GameObject/Apply changes to Prefab &a")]
static void SaveChangesToPrefab()
{
    GameObject[] selection = Selection.gameObjects;
    if (selection.Length < 1) return;
    Undo.RegisterCompleteObjectUndo(selection, "Apply Prefab");
    foreach (GameObject go in selection)
    {
        if (PrefabUtility.GetPrefabType(go) == PrefabType.PrefabInstance)
        {
            PrefabUtility.ReplacePrefab(go, PrefabUtility.GetPrefabParent(go));
        }
    }
}
{% endhighlight %}
### Outro
One of the things I did not mention is on Undo.RegisterCompleteObjectUndo. This is to do a snapshot of the object before we make the change so that we can undo it later if we want. [thebeardphantom](https://gist.github.com/thebeardphantom/ea6362139ee195a8abce){:target="_blank"} uses Undo.RecordObjects. That did not record the snapshot of the prior of some of my tasks(e.g., revert prefab) so I used RegisterCompleteObjectUndo instead. Below is the full code. One last thing i added is the #if (UNITY_EDITOR) so that this script will not get picked up when you build your game. Not sure why, without the #if statement, it will crash the build.
{% highlight csharp%}
#if (UNITY_EDITOR) 
using UnityEngine;
using UnityEditor;
//inspired from https://gist.github.com/thebeardphantom/ea6362139ee195a8abce
public class MyMenuItem : MonoBehaviour
{
    [MenuItem("GameObject/Reset Transform &r")]
    static void ResetTransform()
    {
        GameObject[] selection = Selection.gameObjects;
        if (selection.Length < 1) return;
        Undo.RegisterCompleteObjectUndo(selection, "Zero Position");
        foreach (GameObject go in selection)
        {
            InternalZeroPosition(go);
            InternalZeroRotation(go);
            InternalZeroScale(go);
        }
    }
    [MenuItem("GameObject/Reset Name &n")]
    static void ResetName()
    {
        GameObject[] selection = Selection.gameObjects;
        if (selection.Length < 1) return;
        Undo.RegisterCompleteObjectUndo(selection, "Reset Name");
        foreach (GameObject go in selection)
        {
            Rename(go);
        }
    }
    [MenuItem("GameObject/Revert Prefab &p")]
    static void RevertPrefab()
    {
        GameObject[] selection = Selection.gameObjects;
        if (selection.Length < 1) return;
        Undo.RegisterCompleteObjectUndo(selection, "Revert Prefab");
        foreach (GameObject go in selection)
        {
            if (PrefabUtility.GetPrefabType(go) == PrefabType.PrefabInstance)
            {
                PrefabUtility.RevertPrefabInstance(go);
            }
        }
    }
    [MenuItem("GameObject/Apply changes to Prefab &a")]
    static void SaveChangesToPrefab()
    {
        GameObject[] selection = Selection.gameObjects;
        if (selection.Length < 1) return;
        Undo.RegisterCompleteObjectUndo(selection, "Apply Prefab");
        foreach (GameObject go in selection)
        {
            if (PrefabUtility.GetPrefabType(go) == PrefabType.PrefabInstance)
            {
                PrefabUtility.ReplacePrefab(go, PrefabUtility.GetPrefabParent(go));
            }
        }
    }
    [MenuItem("GameObject/Toggle Lock Inspector &l")]
    static void LockInspector()
    {
        ActiveEditorTracker.sharedTracker.isLocked = !ActiveEditorTracker.sharedTracker.isLocked;
        ActiveEditorTracker.sharedTracker.activeEditors[0].Repaint();
    }
    private static void Rename(GameObject go)
    {
        int start = go.name.IndexOf("(");
        print(start);
        int end = go.name.IndexOf(")");
        print(end);
        if (start != -1 && end != -1 && start < end)
        {
            print("rename");
            go.name = go.name.Substring(0, start);
        }
    }
    private static void InternalZeroPosition(GameObject go)
    {
        go.transform.localPosition = Vector3.zero;
    }
    private static void InternalZeroRotation(GameObject go)
    {
        go.transform.localRotation = Quaternion.Euler(Vector3.zero);
    }
    private static void InternalZeroScale(GameObject go)
    {
        go.transform.localScale = Vector3.one;
    }
}
#endif
{% endhighlight %}