# Tech Academy C# Live Project Code Summary

Introduction
For the end of my C# course with the Tech Academy, I worked on a team to help develop and refine an MVC web application for a theatre troupe. We utilized Scrum methodology in Azure DevOps, which involved daily standups and code retrospectives.

I started the project by familiarizing myself with the code, exploring links, testing what was working, and making note of what needed improvement. My first story on the project involved front-end maintenance, as I created a custom scroll bar to match the color theme of the application. Afterwards, I dove heavily into back-end MVC core principles.

Back-end stories:
1. Creating Models and related Crud pages
2. Creating and stylizing a partial view for a model
3. Using AJAX to make calls to our database
4. Utilizing a Bootstrap modal and AJAX for database entry deletion.


**1. Creating Models and related Crud pages**

I was assigned to the Blog section of the web application. My first core assignment was creating models for a blog's author and a blog's comments.

    public class BlogAuthor
    {
        public int BlogAuthorId { get; set; }       //PK of BlogAuthor
        public string Name { get; set; }            //Name of BlogAuthor
        public string Bio { get; set; }             //Description of BlogAuthor
        public DateTime Joined { get; set; }        //When the BlogAuthor joined
        public DateTime? Left { get; set; }         //When the BlogAuthor left; can be null
    }
    
A relatively simple model, the BlogAuthor was a great way to start the project. I became more familiar with MVC and improved commenting practices. After creating this model, I was able to use Visual Studio to create basic CRUD views, which included Create, Delete, Edit, Details, and Index files.

The other model I creatd was for a blog's comments:

    public class Comment
    {

        public int CommentId { get; set; }              //PK of Comment
        public ApplicationUser Author { get; set; }     //Author of Comment
        public string Message { get; set; }             //Actual text of Comment
        public DateTime CommentDate { get; set; }       //Date of Comment initialization
        public int Likes { get; set; }                  //Number of Likes of Comment
        public int Dislikes { get; set; }               //Number of Dislikes of Comment

        public Comment()
        {
            //The parameters below are the same initially for every Comment.
            CommentDate = DateTime.Now;
            Likes = 0;
            Dislikes = 0;
        }
    }
    
This model became the core of all my future stories. While I created a few basic properties, this model took it a step further by having a few methods, including a constructor. After creating basic CRUD views, I edited the Create.cshtml view to only take in a message property due to the constructor. This made any new comments much easier to make.

**2. Creating and stylizing a partial view for a model**

My next goal was to create an easy way to display all the entries in the Comment model for any View. As such, I created a partial view for the Comment model. 

    @model IEnumerable<TheatreCMS2.Areas.Blog.Models.Comment>

    @foreach (var item in Model)
    {
        @Html.AntiForgeryToken()
        <div class="CommentsSection" id=@($"{"Comment"}{item.CommentId}")>
            <p class="CommentsHeader">
                Author: @Html.DisplayFor(modelItem => item.Author) || @item.TimeDifference()
            </p>
            <p>
                @Html.DisplayFor(modelItem => item.Message)
            </p>
        </div>
        
I started the partial view by displaying the author, the actual message of the comment, and the time since the comment was last posted. The time difference was represented by a simple method in the Comment Model:

        public string TimeDifference()
        {
            TimeSpan timeDifference = DateTime.Now - CommentDate;
            string result = "";
            if (timeDifference.Days > 1)
            {
                result = "Posted " + Convert.ToString(timeDifference.Days) + " days ago";
            }
            else if (timeDifference.Days == 1)
            {
                result = "Posted " + Convert.ToString(timeDifference.Days) + " day ago";
            }
            else if (timeDifference.Hours > 1)
            {
                result = "Posted " + Convert.ToString(timeDifference.Hours) + " hours ago";
            }
            else if (timeDifference.Hours == 1)
            {
                result = "Posted " + Convert.ToString(timeDifference.Hours) + " hour ago";
            }
            else
            {
                result = "Just posted recently";
            }
            return result;
        }
        
This method gave the user an overview of when the comment was last posted without having to sift or interpret a large swath of data.

**3. Using AJAX to make calls to our database**

The next step was displaying the likes, dislikes, and a bar displaying the like to dislike ratio. These values were not only meant to be calls directly to the database, but to be updated through AJAX. 

        <p>
            @*Each like button has an onclick function that calls Incrementer in the script below, passing the model item's CommentId and passing true.
              Also, a unique ID for the like amount span is created, which is equal to "likeAmountX", where X also equals the CommentID.*@
              
            <button class="commentButtons likeButton" onclick="Incrementer(@item.CommentId, true)"><i class="fa fa-thumbs-o-up"></i>
             <span id=@($"{"likeAmount"}{@item.CommentId.ToString()}")>@Html.DisplayFor(modelItem => item.Likes)</span></button>  ||
             
            @*Similarly, each dislike button has an onclick function that calls Incrementer in the script below, passing the model item's CommentId and passing false.
              A unique ID for the dislike amount span is created, which is equal to "dislikeAmountX", where X equals the CommentID.*@
              
            <button class="commentButtons dislikeButton" onclick="Incrementer(@item.CommentId, false)"><i class="fa fa-thumbs-down"></i>
             <span id=@($"{"dislikeAmount"}{@item.CommentId.ToString()}")>@Html.DisplayFor(modelItem => item.Dislikes)</span></button>  ||
            <button class="commentButtons replyButton"><i class="fa fa-mail-reply"></i> Reply</button>
        </p>
        <div class="progress likeRatioBar">
            @*For each comment two progress bars are created: The first shows the like ratio, the second shows the dislike ratio. Both are in the same 'progress' class.
              We create a unique id for each like and dislike bar, which is based off of the item's ID. We also use inline styling to set the width of the bars, as 
              that is how the bootsrap class functions. The width is based off of an AJAX callback that is called after the Incrementer() function, which ensures
              that the LikeRatio() for the item has been updated. The like bar is set to the likeRatio, whereas the dislike bar is set to the inverse, ensuring 
              that both bars fill up 100% of their container.*@
              
            <div class="progress-bar bg-info" id=@($"{"likeBar"}{@item.CommentId.ToString()}") role="progressbar" style="width: @string.Concat(Convert.ToString(item.LikeRatio()*100),"%")" aria-valuenow="@Convert.ToString(item.LikeRatio()*100)" aria-valuemin="0" aria-valuemax="100"></div>
            <div class="progress-bar bg-danger" id=@($"{"dislikeBar"}{@item.CommentId.ToString()}") role="progressbar" style="width: @string.Concat(Convert.ToString((1-item.LikeRatio())*100),"%")" aria-valuenow="@Convert.ToString((1-item.LikeRatio())*100)" aria-valuemin="0" aria-valuemax="100"></div>
        </div>
        <p>
            <button class="commentButtons deleteButton" data-toggle="modal" data-target=@($"{"#commentModal"}{item.CommentId.ToString()}")><i class="fa fa-trash"></i> Delete</button>
        </p>
        
The code above was in the foreach loop, meaning that the like buttons, the dislike buttons, and the like/dislike bars each had their own unique ID, making it easier to grab each item with Javascript. These unique IDs were based off of the CommentID in the database. Any functions called (typically by buttons pressed) would pass the CommentID as a paramter, making sure that one function could handle each database instance.

The first function was Incrementer, which would increase the likes or dislikes of a Comment through AJAX.

    //This function is called upon button click. The parameters are the ID of the respective item
    //and a bool, where true represents likes and false represents dislikes. If called from a like button,
    //the bool is true. If called from a dislike button, the bool is false.
    function Incrementer(incrementedID, boolValue) {
        var xhttp = new XMLHttpRequest();
        //We change the URL from CommentsController.cs that we call depending if we are increasing likes or dislikes.
        if (boolValue) {
            var pathway = "/Comments/IncreaseLikes/" + incrementedID;
        }
        else {
            var pathway = "/Comments/IncreaseDislikes/" + incrementedID;
        }
        xhttp.onreadystatechange = function () {
            if (this.readyState == 4 && this.status == 200) {
                if (boolValue) {
                    //If called from the Like button, we increase the amount of likes for the item, then return 
                    //the new amount of likes to the respective space.
                    document.getElementById("likeAmount".concat(incrementedID)).innerHTML = this.responseText;
                }
                else {
                    //If called from the Dislike button, we increase the amount of dislikes for the item, then return 
                    //the new amount of dislikes to the respective space.
                    document.getElementById("dislikeAmount".concat(incrementedID)).innerHTML = this.responseText;
                }
                //Once the database is updated, we'll perform a callback with the likeRatioBarsUpdate function to change the likeRatio bar.
                likeRatioBarsUpdate(incrementedID);
            }
        };      
        xhttp.open("POST", pathway, true);
        xhttp.send();
    }

As seen above, this function would take in the CommentID and a boolean. 
