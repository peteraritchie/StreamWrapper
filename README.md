StreamWrapper
=============

StreamWrapper is a simple class that implements `Stream` and wraps a `Stream` object and is a simple pass-through wrapper.  `StreamWrapper` is entended to wrap a `Stream` object so that the lifespan of that `Stream` object can be matched to the lifespan of another.

For example, you might create an `IsolatedStorageFileStream` object from an `IsolatedStorageFile` object:

```C#
    private static Stream GetStream()
	{
		var storage =
			IsolatedStorageFile.GetStore(
				IsolatedStorageScope.Application | IsolatedStorageScope.Assembly | IsolatedStorageScope.Domain, null, null);
		return storage.CreateFile("file.ext");
	}
  
  //...
  
  using(Stream stream = GetStream())
  {
    //...
  }
```

But, now you have a dangling `IsolatedStorageFile` that you can't deterministically dispose of.  The GC will eventually deal with it; but only on its schedule.  `StreamWrapper` allows you derive a new class to wrap this stream and pair it with an `IsolatedStorageFile` object so that they can be disposed at the same time.  For example:

```C#
    public class IsolatedStorageFileStreamWrapper : StreamWrapper
	{
		private readonly IsolatedStorageFile file;

		public IsolatedStorageFileStreamWrapper(IsolatedStorageFile file, IsolatedStorageFileStream stream) : base(stream)
		{
			this.file = file;
		}

		protected override void Dispose(bool disposing)
		{
			// Dispose Stream
			base.Dispose(disposing);
			if (!disposing) return;
			// Dispose file
			using (file)
			{
			}
		}
	}
```

Then you can simply wrap both objects with the new `IsolatedStorageFileStreamWrapper` so that when the code using the `Stream` also disposes the `IsolatedStoragefile` object:

```C#
    private static Stream GetStream()
  {
		var storage =
			IsolatedStorageFile.GetStore(
				IsolatedStorageScope.Application | IsolatedStorageScope.Assembly | IsolatedStorageScope.Domain, null, null);
		return new IsolatedStorageFileStreamWrapper(storage, storage.CreateFile("file.ext"));
	}
  
  //...
  
  using(Stream stream = GetStream())
  {
    //...
  }
```
