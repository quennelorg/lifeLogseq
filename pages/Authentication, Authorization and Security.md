- [[Authentication, Authorization and Security/modelUser]]
- [[Authentication, Authorization and Security/Managing Passwords]]
- [[Authentication, Authorization and Security/How Authentication with JWT Works]]
- Using jwt
  collapsed:: true
	- ```javascript
	  exports.signup = catchAsync(async (req, res, next) => {
	    const newUser = await User.create({
	      name: req.body.name,
	      email: req.body.email,
	      password: req.body.password,
	      passwordConfirm: req.body.passwordConfirm,
	    });
	  
	    const token = jwt.sign({ id: newUser._id }, process.env.JWT_SECRET, {
	      expiresIn: process.env.JWT_EXPIRES_IN,
	    });
	  
	    res.status(201).json({
	      status: 'success',
	      token,
	      data: {
	        user: newUser,
	      },
	    });
	  });
	  ```
- login
  collapsed:: true
	- model
	  collapsed:: true
		- ```javascript
		  userSchema.methods.correctPassword = async function (
		    candidatePassword,
		    userPassword,
		  ) {
		    return bcrypt.compare(candidatePassword, userPassword);
		  };
		  ```
	- authController
	  collapsed:: true
		- ```javascript
		  const signToken = (id) => {
		    return jwt.sign({ id }, process.env.JWT_SECRET, {
		      expiresIn: process.env.JWT_EXPIRES_IN,
		    });
		  };
		  
		  exports.signup = catchAsync(async (req, res, next) => {
		    const newUser = await User.create({
		      name: req.body.name,
		      email: req.body.email,
		      password: req.body.password,
		      passwordConfirm: req.body.passwordConfirm,
		    });
		  
		    const token = signToken(newUser._id);
		  
		    res.status(201).json({
		      status: 'success',
		      token,
		      data: {
		        user: newUser,
		      },
		    });
		  });
		  exports.login = catchAsync(async (req, res, next) => {
		    const { email, password } = req.body;
		  
		    // 1 check email and password exist
		    if (!email || !password) {
		      next(new AppError('Please provide email and password!', 400));
		    }
		  
		    // 2 check if user exists && password is correct
		  
		    const user = await User.findOne({ email: email }).select('+password');
		    const correct = await user.correctPassword(password, user.password);
		  
		    if (!user || !correct) {
		      return next(new AppError('Incorrect email or password', 401));
		    }
		  
		    // 3 if everything ok, send token to client
		    const token = signToken(user._id);
		    res.status(200).json({
		      status: 'success',
		      token,
		    });
		  });
		  ```
- Protecting Tour Routes
  collapsed:: true
	- ```javascript
	  router
	    .route('/')
	    .get(authController.protect, getAllTours)
	    .post(authController.protect, createTour);
	  
	  exports.protect = catchAsync(async (req, res, next) => {
	    // 1 getting token and check of its there
	    let token;
	    if (
	      req.headers.authorization &&
	      req.headers.authorization.startsWith('Bearer')
	    ) {
	      token = req.headers.authorization.split(' ')[1];
	    }
	    if (!token) {
	      return next(
	        new AppError('You are not logged in! Please log in to get access.', 400),
	      );
	    }
	    // 2 validate token
	    const decoded = await promisify(jwt.verify)(token, process.env.JWT_SECRET);
	  
	    // 3 check if user still exists
	    const currentUser = await User.findById(decoded.id);
	    if (!currentUser) {
	      return next(
	        new AppError(
	          'The token belonging to this user does not longer exist.',
	          401,
	        ),
	      );
	    }
	  
	    // 4 check if use changed password after the token was issued
	    if (currentUser.changedPasswordAfter(decoded.iat)) {
	      return next(
	        new AppError('User recently changed password! Please log in again.', 401),
	      );
	    }
	  
	    // Grant access to protect route
	    req.user = currentUser;
	    next();
	  });
	  ```
- Authorization: User Roles and Permissions
  collapsed:: true
	- usermodel
	  collapsed:: true
		- ```javascript
		    role: {
		      type: String,
		      enum: ['user', 'guide', 'lead-guide', 'admin'],
		      default: 'user',
		    },
		  ```
	- route
	  collapsed:: true
		- ```javascript
		  router
		    .route('/:id')
		    .get(getTour)
		    .patch(updateTour)
		    .delete(
		      authController.protect,
		      authController.restrictTo('admin', 'lead-guide'),
		      deleteTour,
		    );
		  ```
	- restrictTo
	  collapsed:: true
		- ```javascript
		  exports.restrictTo = (...roles) => {
		    return (req, res, next) => {
		      if (!roles.includes(req.user.role)) {
		        return next(
		          new AppError('You do not have permission to perform this action', 403),
		        );
		      }
		      next();
		    };
		  };
		  ```
- Password Reset Functionality: Reset Token
  collapsed:: true
	- authcontroller
	  collapsed:: true
		- ```javascript
		  exports.forgotPassword = catchAsync(async (req, res, next) => {
		    // 1 get user based on posted email
		    const user = await User.findOne({ email: req.body.email });
		    if (!user) {
		      return next(new AppError('There is no user with email address', 404));
		    }
		    // 2 generate the random reset token
		    const resetToken = user.createPasswordResetToken();
		    await user.save({ validateBeforeSave: false });
		    // 3 send it to user's email
		  });
		  ```
	- userModel
	  collapsed:: true
		- ```javascript
		  userSchema.methods.createPasswordResetToken = function () {
		    const resetToken = crypto.randomBytes(32).toString('hex');
		  
		    this.passwordResetToken = crypto
		      .createHash('sha256')
		      .update(resetToken)
		      .digest('hex');
		    console.log({ resetToken }, this.passwordResetToken);
		    this.passwordResetExpires = Date.now() + 10 * 60 * 1000;
		  
		    return resetToken;
		  };
		  ```
- Sending Emails with Nodemailer
  collapsed:: true
	- https://mailtrap.io/inboxes
	- email.js
	  collapsed:: true
		- ```javascript
		  const nodemailer = require('nodemailer');
		  
		  const sendEmail = async (options) => {
		    // 1 create a transporter
		    // const transporter = nodemailer.createTransport({
		    //   service: 'Gmail',
		    //   auth: {
		    //     user: process.env.EMAIL_USERNAME,
		    //     pass: process.env.EMAIL_PASSWORD,
		    //   },
		    // });
		    // activate in gmail 'less secure app' option
		    const transport = nodemailer.createTransport({
		      host: process.env.EMAIL_HOST,
		      port: process.env.EMAIL_PORT,
		      auth: {
		        user: process.env.EMAIL_USERNAME,
		        pass: process.env.EMAIL_PASSWORD,
		      },
		    });
		  
		    // 2 Define the email options
		  
		    const mailOptions = {
		      from: 'QuennelCoder',
		      to: options.email,
		      subject: options.subject,
		      text: options.message,
		      // html: ,
		    };
		    // 3 actually send the email
		    await transport.sendMail(mailOptions);
		  };
		  
		  module.exports = sendEmail;
		  
		  ```
	- config
	  collapsed:: true
		- ```javascript
		  EMAIL_USERNAME=f3985c59609ccd
		  EMAIL_PASSWORD=e894dfd26a3560
		  EMAIL_HOST=sandbox.smtp.mailtrap.io
		  EMAIL_PORT=2525
		  ```
	- forgotPassword
	  collapsed:: true
		- ```javascript
		  exports.forgotPassword = catchAsync(async (req, res, next) => {
		    // 1 get user based on posted email
		    const user = await User.findOne({ email: req.body.email });
		    if (!user) {
		      return next(new AppError('There is no user with email address', 404));
		    }
		    // 2 generate the random reset token
		    const resetToken = user.createPasswordResetToken();
		    await user.save({ validateBeforeSave: false });
		    // 3 send it to user's email
		    const resetURL = `${req.protocol}://${req.get('host')}/api/v1/users/resetPassword/${resetToken}`;
		  
		    const message = `Forgot your password? Submit a Patch request with your new password and passwordConfirm to: ${resetURL}.\n If you didn't forget your password, please ignore this email!`;
		  
		    try {
		      await sendEmail({
		        email: user.email,
		        subject: 'Your Password reset token (valid for 10 min)',
		        message,
		      });
		  
		      res.status(200).json({
		        status: 'success',
		        message: 'Token sent to email!',
		      });
		    } catch (err) {
		      user.passwordResetToken = undefined;
		      user.passwordResetExpires = undefined;
		      await user.save({ validateBeforeSave: false });
		      return next(
		        new AppError(
		          'There was an error sending the email, Try again later!',
		          500,
		        ),
		      );
		    }
		  });
		  ```
- Password Reset Functionality: Setting New Password
  collapsed:: true
	- useModel
	  collapsed:: true
		- ```javascript
		  userSchema.pre('save', function (next) {
		    if (!this.isModified('password') || this.isNew) return next();
		  
		    this.passwordChangedAt = Date.now() - 1000;
		    next();
		  });
		  ```
	- authController
	  collapsed:: true
		- ```javascript
		  exports.resetPassword = catchAsync(async (req, res, next) => {
		    // 1 get user based on the token
		    const hashedToken = crypto
		      .createHash('sha256')
		      .update(req.params.token)
		      .digest('hex');
		  
		    const user = await User.findOne({
		      passwordResetToken: hashedToken,
		      passwordResetExpires: { $gt: Date.now() },
		    });
		    // 2 if token has not expired, and there is user, set the new password
		    if (!user) {
		      return next(new AppError('Token is invalid or has expired', 400));
		    }
		  
		    user.password = req.body.password;
		    user.passwordConfirm = req.body.passwordConfirm;
		    user.passwordResetToken = undefined;
		    user.passwordResetExpires = undefined;
		    await user.save();
		    // 3 update changedPasswordAt property for the user
		    // 4 log the user in, send jwt
		    const token = signToken(user._id);
		  
		    res.status(201).json({
		      status: 'success',
		      token,
		    });
		  });
		  ```
- Updating the Current User: Password