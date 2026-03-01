# Kirkwood-RP
Every holiday update every taco Tuesday every using UnityEngine;

public class Builder : MonoBehaviour
{
    public GameObject wallPrefab; // The item you want to build

    void Update()
    {
        // If the player clicks the left mouse button
        if (Input.GetMouseButtonDown(0))
        {
            BuildObject();
        }
    }

    void BuildObject()
    {
        // This creates the wall at the current script's position
        Instantiate(wallPrefab, transform.position, transform.rotation);
    }
}



