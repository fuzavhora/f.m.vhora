import React, { useState } from 'react';

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
