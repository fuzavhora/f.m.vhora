jiimport React, { useState } from 'react';

const ImageUpload = () => {
  const [images, setImages] = useState([]);

  const handleImageChange = (e) => {
    const files = Array.from(e.target.files);

    const readers = files.map(file => {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.onerror = reject;
        reader.readAsDataURL(file);
      });
    });

    Promise.all(readers).then((imageData) => {
      const newImages = [...images, ...imageData];
      setImages(newImages);
      localStorage.setItem('galleryImages', JSON.stringify(newImages));
    });
  };

  return (
    <div>
      <h2>Upload Images</h2>
      <input type="file" accept="image/*" multiple onChange={handleImageChange} />
      <div style={{ display: 'flex', flexWrap: 'wrap', marginTop: '10px' }}>
        {images.map((img, index) => (
          <img
            key={index}
            src={img}
            alt={`thumbnail-${index}`}
            style={{ width: 100, height: 100, objectFit: 'cover', margin: 5, borderRadius: 5 }}
          />
        ))}
      </div>
    </div>
  );
};

export default ImageUpload;




import React, { useState, useEffect } from 'react';

const ImageUploadWithAlbum = () => {
  const [albums, setAlbums] = useState([]);
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const [category, setCategory] = useState('');
  const [selectedFiles, setSelectedFiles] = useState([]);
  const [filterText, setFilterText] = useState('');

  // Load albums from localStorage
  useEffect(() => {
    const stored = JSON.parse(localStorage.getItem('galleryAlbums')) || [];
    setAlbums(stored);
  }, []);

  // Handle image selection
  const handleFileChange = (e) => {
    setSelectedFiles(Array.from(e.target.files));
  };

  // Convert files to base64
  const convertToBase64 = (files) => {
    const promises = files.map((file) => {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.onerror = reject;
        reader.readAsDataURL(file);
      });
    });
    return Promise.all(promises);
  };

  // Save album
  const handleSubmit = async (e) => {
    e.preventDefault();

    if (!title || !description || !category || selectedFiles.length === 0) {
      alert('Please fill all fields and select images');
      return;
    }

    const imageBase64s = await convertToBase64(selectedFiles);

    const newAlbum = {
      title,
      description,
      category,
      images: imageBase64s,
    };

    const updatedAlbums = [...albums, newAlbum];
    setAlbums(updatedAlbums);
    localStorage.setItem('galleryAlbums', JSON.stringify(updatedAlbums));

    // Reset form
    setTitle('');
    setDescription('');
    setCategory('');
    setSelectedFiles([]);
  };

  // Filtered albums based on title or category
  const filteredAlbums = albums.filter((album) =>
    album.title.toLowerCase().includes(filterText.toLowerCase()) ||
    album.category.toLowerCase().includes(filterText.toLowerCase())
  );

  return (
    <div style={{ padding: '20px' }}>
      <h2>Create Album</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Album Title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          required
          style={{ display: 'block', marginBottom: 10 }}
        />
        <textarea
          placeholder="Album Description"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          required
          style={{ display: 'block', marginBottom: 10 }}
        />
        <input
          type="text"
          placeholder="Category"
          value={category}
          onChange={(e) => setCategory(e.target.value)}
          required
          style={{ display: 'block', marginBottom: 10 }}
        />
        <input
          type="file"
          accept="image/*"
          multiple
          onChange={handleFileChange}
          required
          style={{ display: 'block', marginBottom: 10 }}
        />
        <button type="submit">Add Album</button>
      </form>

      <hr style={{ margin: '30px 0' }} />

      {/* Filter Input */}
      <div style={{ marginBottom: '20px' }}>
        <input
          type="text"
          placeholder="Search by title or category..."
          value={filterText}
          onChange={(e) => setFilterText(e.target.value)}
          style={{
            padding: '8px',
            width: '250px',
            borderRadius: '5px',
            border: '1px solid #ccc'
          }}
        />
      </div>

      <h3>Gallery Albums</h3>
      {filteredAlbums.length === 0 && <p>No albums found.</p>}

      {filteredAlbums.map((album, index) => (
        <div key={index} style={{ marginBottom: '40px' }}>
          <h4>{album.title}</h4>
          <p><strong>Description:</strong> {album.description}</p>
          <p><strong>Category:</strong> {album.category}</p>
          <div style={{ display: 'flex', flexWrap: 'wrap', gap: '10px' }}>
            {album.images.map((img, i) => (
              <img
                key={i}
                src={img}
                alt={`album-${index}-img-${i}`}
                style={{ width: 120, height: 120, objectFit: 'cover', borderRadius: 6 }}
              />
            ))}
          </div>
        </div>
      ))}
    </div>
  );
};

export default ImageUploadWithAlbum;
