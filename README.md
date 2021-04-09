# Prosper IT Consulting C# Live Project Code Summary

**Introduction**

For the end of my C# course with the Tech Academy, I worked with on a team with Prosper IT Consulting to help develop and refine an MVC web application for a theatre troupe. We utilized Scrum methodology in Azure DevOps, which involved daily standups and code retrospectives.

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

As seen above, this function would take in the CommentID and a boolean boolValue. If boolValue is true, a Comment's like was affected. If boolValue is false, a Comment's dislike was affected. This worked by changing the pathway called through our XMLHttpRequest, as I created two different methods in our Comment controller. 

        //POST method for increasing a Comment's likes and returning the new value
        [HttpPost]
        public int IncreaseLikes(int id)
        {
            Comment comment = db.Comments.Find(id);
            comment.Likes += 1;
            db.SaveChanges();
            return comment.Likes;
        }

        //POST method for increasing a Comment's dislikes and returning the new value
        [HttpPost]
        public int IncreaseDislikes(int id)
        {
            Comment comment = db.Comments.Find(id);
            comment.Dislikes += 1;
            db.SaveChanges();
            return comment.Dislikes;
        }

If the server status and the ready state both respond positively, the like/dislike amount is increased in the database and the new value is returned to the view. This returned value is then displayed near the respective button, causing the likes or dislikes to increase without the page having to be refreshed. 

At the end of the Incrementer function, a callback occurs to the likeRatioBarsUpdate() function, another AJAX function that dynamically updates a Bootstrap progress bar that shows a comparison of likes to dislikes.

        <div class="progress likeRatioBar">
            @*For each comment two progress bars are created: The first shows the like ratio, the second shows the dislike ratio. Both are in the same 'progress' class.
              We create a unique id for each like and dislike bar, which is based off of the item's ID. We also use inline styling to set the width of the bars, as 
              that is how the bootsrap class functions. The width is based off of an AJAX callback that is called after the Incrementer() function, which ensures
              that the LikeRatio() for the item has been updated. The like bar is set to the likeRatio, whereas the dislike bar is set to the inverse, ensuring 
              that both bars fill up 100% of their container.*@
              
            <div class="progress-bar bg-info" id=@($"{"likeBar"}{@item.CommentId.ToString()}") role="progressbar" style="width: @string.Concat(Convert.ToString(item.LikeRatio()*100),"%")" aria-valuenow="@Convert.ToString(item.LikeRatio()*100)" aria-valuemin="0" aria-valuemax="100"></div>
            <div class="progress-bar bg-danger" id=@($"{"dislikeBar"}{@item.CommentId.ToString()}") role="progressbar" style="width: @string.Concat(Convert.ToString((1-item.LikeRatio())*100),"%")" aria-valuenow="@Convert.ToString((1-item.LikeRatio())*100)" aria-valuemin="0" aria-valuemax="100"></div>
        </div>
        
This progress bar is filled by two factors: a likeRatio(), which is a value from 0-1, and 1-likeRatio(), which represents the opposite value. Together, these both values add up to 1. When each value is multiplied by 100 and given a % sign, they cover 100%, meaning the entire likeRatioBar is filled. These values are both updated through the likeRatioBarsUpdate() function.

    //This function is a callback that is completed once Incrementer() is finished, ensuring that LikeRatio() has been updated.
    function likeRatioBarsUpdate(incrementedID) {
        var pathway = "/Comments/LikeRatio/" + incrementedID;
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function () {
            if (this.readyState == 4 && this.status == 200) {
                //Using the new LikeRatio(), we set the width of the like bar equal to the percentage form of the LikeRatio()'s returned value
                //and the dislike bar equal to the inverse, ensuring that 100% is met.
                document.getElementById("likeBar" + incrementedID).style.width = String(this.responseText * 100) + "%";
                document.getElementById("dislikeBar" + incrementedID).style.width = String((1-this.responseText) * 100) + "%";
            }
        }

      xhttp.open("GET", pathway, true);
      xhttp.send();
    }
    
This is another AJAX function (that is only called AFTER the respective likes/dislikes of a Comment are increased) that dynamically updates the like and dislike bars in the progress bar container. The function calls the likeRatio for the selected Comment and returns that value. This returned value is then used for setting the width of the likeBar, and 1 minues the returned value is used for setting the width of the dislikeBar. This results in the progress bar dynamically updating after each like and dislike while always filling up 100% of the container.

**4. Utilizing a Bootstrap modal and AJAX for database entry deletion.**

Finally, I impletmenedted a modal to be displayed any time a delete button is clicked.

    <div class="modal fade" id=@($"{"commentModal"}{@item.CommentId.ToString()}") tabindex="-1" role="dialog" aria-labelledby="commentModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content commentDeleteModal">
                <div class="modal-header">
                <h5 class="modal-title" id="exampleModalLabel">Comment Deletion</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
              <div class="modal-body">
                  Are you sure you want to delete the comment below? This cannot be undone! 
                  <br />
                  <br />
                  @item.Message
              </div>
              <div class="modal-footer">
                  <button type="button" class="btn btn-primary" data-dismiss="modal">Close</button>
                  <button type="button" class="btn btn-danger" onclick="deleteComment(@item.CommentId)" data-dismiss="modal" data-target="#commentDeleteMessage">Yes, I'm sure</button>
              </div>
            </div>
        </div>
    </div>
    
This modal would check with the user if they were positive they wanted to delete the comment. If the user clicks elsewhere or clicked the no button, the modal would close. Othewrise, the modal would call a deleteComment function, also done through AJAX.

    //This function will delete the relevant comment from the db, then hide all content associated with that comment.
    function deleteComment(commentID) {
        var pathway = "/Comments/DeleteWithoutRedirect/" + commentID;
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function () {
            if (this.readyState == 4 && this.status == 200) {
                document.getElementById("Comment" + commentID).style.display = "none";
                document.getElementById("commentDeleteMessage").classList.add("commentDeleteMagic");
                document.getElementById("commentDeleteMessage").style.display = "block";
                //We set a timeout of 4 seconds on this callback so that the message can fade in, display for 3 seconds, then fade out
                setTimeout("removeClass()", 4000);

            }
        }
        xhttp.open("POST", pathway, true);
        xhttp.send();
    }
    
If the delete call is successful, we would set the display of the removed comment to none. Then, we would add the class "commentDeleteMagic" to a delete message we had on the page. 

    .commentDeleteMagic {
        position: fixed;
        top: 110px;
        height: 50px;
        display: block;
        color: var(--light-color);
        z-index: 10;
        width: 100%;
        max-width: 1140px;
        text-align: center;
        border-radius: 20px;
        opacity: 0;
        animation-name: commentFade;
        animation-duration: 4s;
    }

With this class being added, a green dialog box would fade in to view, letting know the comment had been deleted. The message would last for 3 seconds in view, then fade away. This is tied back to our deleteComment function, as there is a callback at the end of the function to removeClass()

    //A simple callback removing the class from our commentDeleteMessage div, causing us to delete something else and the message to appear again.
    function removeClass() {
        document.getElementById("commentDeleteMessage").style.display = "none";
        document.getElementById("commentDeleteMessage").classList.remove("commentDeleteMagic");
    }
    
This would remove the class that was previously added to the deletion confirmation message, meaning that if another entry is deleted, the message would display once more, as the animation would reactivate.

**Conclusion**

Overall, the project was a great experience. I became more familiar with Agile and Scrum, as I was part of a team that all worked together on different parts of the project. I gained valuable experience with ASP.NET MVC, as I created models, views, and a controller. I practiced with a little bit of front-end work, frequently utilizing bootstrap for my views. Finally, I practiced with AJAX, making XMLHttpRequests to asynchrously update databases and views.
