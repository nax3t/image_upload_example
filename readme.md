# Image Upload Instructions
Video tutorial available [here](https://youtu.be/RHd4rP9U9SA)
1. Sign up for [cloudinary](https://cloudinary.com/)
    - Don't forget to check your email and activate your account
2. Install multer and cloudinary into your project
    - `npm i -S multer cloudinary`
3. Update /campgrounds/new.ejs
```HTML
<% include ../partials/header %>
    <div class="row">
        <h1 style="text-align: center">Create a New Campground</h1>
        <div style="width: 30%; margin: 25px auto;">
            <form action="/campgrounds" method="POST" enctype="multipart/form-data">
                <div class="form-group">
                    <input class="form-control" type="text" name="campground[name]" placeholder="name">
                </div>
                <div class="form-group">
                    <label for="image">Image</label>
                    <input type="file" id="image" name="image" accept="image/*" required>
                </div>
                <div class="form-group">
                    <input class="form-control" type="text" name="campground[description]" placeholder="description">
                </div>
                <div class="form-group">
                    <button class="btn btn-lg btn-primary btn-block">Submit!</button>
                </div>
            </form>
            <a href="/campgrounds">Go Back</a>
        </div>
    </div>
<% include ../partials/footer %>


```
5. Add multer and cloudinary configuration code to your /routes/campgrounds.js file:
```JS
var multer = require('multer');
var storage = multer.diskStorage({
  filename: function(req, file, callback) {
    callback(null, Date.now() + file.originalname);
  }
});
var imageFilter = function (req, file, cb) {
    // accept image files only
    if (!file.originalname.match(/\.(jpg|jpeg|png|gif)$/i)) {
        return cb(new Error('Only image files are allowed!'), false);
    }
    cb(null, true);
};
var upload = multer({ storage: storage, fileFilter: imageFilter})

var cloudinary = require('cloudinary');
cloudinary.config({ 
  cloud_name: 'learntocodeinfo', 
  api_key: process.env.CLOUDINARY_API_KEY, 
  api_secret: process.env.CLOUDINARY_API_SECRET
});
```
6. Replace `cloud_name` with your cloud name from the cloudinary dashboard (after logging in to the account you created). 
7. Replace `api_key` and `api_secret` with your api key and secret from the cloudinary dashboard (use [ENV variables](https://github.com/motdotla/dotenv) to keep them secure)
8. Add upload.single('image') to your POST route's middleware chain:
```JS
router.post("/", middleware.isLoggedIn, upload.single('image'), function(req, res) {
```
9. Add the following cloudinary code to your POST route:
```JS
cloudinary.uploader.upload(req.file.path, function(result) {
  // add cloudinary url for the image to the campground object under image property
  req.body.campground.image = result.secure_url;
  // add author to campground
  req.body.campground.author = {
    id: req.user._id,
    username: req.user.username
  }
  Campground.create(req.body.campground, function(err, campground) {
    if (err) {
      req.flash('error', err.message);
      return res.redirect('back');
    }
    res.redirect('/campgrounds/' + campground.id);
  });
});
```
10. If you're using Google maps then you'll need to put all of the cloudinary code from above inside of the geocoder.geocode() callback. If you're not using Google maps then you can ignore this step.
11. I have only included code for creating campgrounds, you'll have to implement editing campgrounds on your own (for now)
