Below is an example Jest‐based test suite covering the small “pure” helper functions in your DirectoryEditor.tsx.  You can expand it to cover the rest of your logic (drag & drop, CRUD, etc.) in a similar way, or add React Testing Library tests against the rendered component if you wish.

1) First, install Jest (and ts‐jest if you use TypeScript):

```bash
npm install --save-dev jest ts-jest @types/jest
npx ts-jest config:init
```

2) Add a script to your package.json:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```

3) Create a test file alongside your DirectoryEditor.tsx—for example `DirectoryEditor.helpers.test.ts`—and paste in the tests below.

```typescript
// DirectoryEditor.helpers.test.ts
import {
  findNodeById,
  getNextAvailableName,
  collectDeletedItems,
  createRecycleBinFolder
} from '../src/components/DirectoryEditor'; // adjust path as needed
import { DocType, type DirectoryItem, type Folder, type Document } from '../src/store/DirectoryItem';

describe('DirectoryEditor helper functions', () => {

  const leafDoc = (id: string, name: string, deleted = false): Document => ({
    id,
    name,
    docType: DocType.REGULATION,
    keywords: [],
    sortOrder: 0,
    sortBy: 'DEFAULT',
    fileName: name + '.txt',
    fileExt: '.txt',
    isBright: false,
    isVisible: true,
    isModified: false,
    isFillableForm: false,
    hasMoved: false,
    hideInUI: false,
    usedInAI: false,
    includeInQuestions: true,
    isDeleted: deleted,
    hasEditorChange: false,
    deletedOn: null,
    modifiedOn: null,
    modifiedBy: null,
    createdOn: null,
    createdBy: null,
    path: '',
    pathRealtime: '',
    originalName: '',
    originalPath: '',
    previousPath: '',
    currentPath: '',
    vc: 1,
    link: ''
  } as unknown as Document);

  const makeFolder = (
    id: string,
    name: string,
    items: DirectoryItem[],
    deleted = false
  ): Folder => ({
    id,
    name,
    docType: DocType.FOLDER,
    keywords: [],
    isOpen: false,
    isOpenDefault: false,
    path: '',
    pathRealtime: '',
    isVisible: true,
    hideInUI: false,
    fileCount: 0,
    folderCount: 0,
    totalCount: 0,
    totalFileCount: 0,
    totalFolderCount: 0,
    items,
    sortOrder: 0,
    isDeleted: deleted,
    hasEditorChange: false,
    deletedOn: null,
    modifiedOn: null,
    modifiedBy: null,
    createdOn: null,
    createdBy: null,
    originalPath: '',
    previousPath: '',
    currentPath: '',
    show() {},
    hide() {},
    open() {},
    close() {},
    toggleOpen() {},
    getFolderPath: async () => '',
    hasEditorChange: false
  } as unknown as Folder);

  describe('getNextAvailableName', () => {
    it('returns base name if not taken', () => {
      const items = [{ name: 'Foo' }, { name: 'Bar' }] as any[];
      expect(getNextAvailableName(items, 'New Folder')).toBe('New Folder');
    });

    it('appends (1), (2), … if names collide', () => {
      const items = [
        { name: 'New Folder' },
        { name: 'New Folder (1)' },
        { name: 'Something else' }
      ] as any[];
      expect(getNextAvailableName(items, 'New Folder')).toBe('New Folder (2)');
    });
  });

  describe('collectDeletedItems', () => {
    it('finds all deleted items recursively', () => {
      const tree = [
        leafDoc('d1', 'A', true),
        leafDoc('d2', 'B', false),
        makeFolder('f1', 'Folder1', [
          leafDoc('d3', 'C', true),
          makeFolder('f2', 'Folder2', [
            leafDoc('d4', 'D', true),
            leafDoc('d5', 'E', false)
          ])
        ])
      ];

      const deleted = collectDeletedItems(tree);
      // should contain d1, d3, d4
      const ids = deleted.map((i) => i.id).sort();
      expect(ids).toEqual(['d1', 'd3', 'd4']);
    });
  });

  describe('createRecycleBinFolder', () => {
    it('builds a synthetic folder with correct counts', () => {
      const deletedMock = [
        leafDoc('x1', 'X', true),
        makeFolder('y1', 'Y', [], true),
        leafDoc('x2', 'Z', true)
      ];

      const bin = createRecycleBinFolder(deletedMock);
      expect(bin.id).toBe('recycle-bin');
      expect(bin.name).toBe('Recycle Bin');
      // 2 files, 1 folder
      expect(bin.fileCount).toBe(2);
      expect(bin.folderCount).toBe(1);
      expect(bin.totalCount).toBe(3);
      // its items are exactly the deletedMock array
      expect(bin.items).toEqual(deletedMock);
    });
  });

  describe('findNodeById', () => {
    const nestedTree = [
      makeFolder('root', 'Root', [
        leafDoc('d1', 'D1'),
        makeFolder('sub', 'Sub', [
          leafDoc('d2', 'D2'),
        ]),
      ])
    ];

    it('finds top‐level folder by id', () => {
      const found = findNodeById(nestedTree, 'root');
      expect(found).not.toBeNull();
      expect(found!.id).toBe('root');
    });

    it('finds nested document by id', () => {
      const found = findNodeById(nestedTree, 'd2');
      expect(found).not.toBeNull();
      expect(found!.name).toBe('D2');
    });

    it('returns null if not found', () => {
      expect(findNodeById(nestedTree, 'xxx')).toBeNull();
    });
  });

  describe('findNodeById special recycle‐bin', () => {
    it('returns a Recycle Bin when id==="recycle-bin" and instance passed', () => {
      // simulate a BrightSource instance with a Directory.allItems
      const fakeBrightSource = {
        Directory: {
          allItems: [
            leafDoc('a', 'A', true),
            leafDoc('b', 'B', false)
          ]
        }
      };
      const got = findNodeById([], 'recycle-bin', fakeBrightSource as any)!;
      expect(got.id).toBe('recycle-bin');
      // should have only 1 deleted item
      expect((got as Folder).items.length).toBe(1);
    });
  });
});
```

–––

If you want to test the actual React component (DirectoryEditor.tsx) and its UI behaviors (drill down, create folder, delete, etc.), you can install React Testing Library and write tests like:

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom
```

Then in `DirectoryEditor.render.test.tsx`:

```tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrightSourceTree } from '../src/components/DirectoryEditor';
import { BrightSource } from '../src/store/BrightSource';
import { Tabs } from '../src/store/Tabs';
import { ChatAppContext } from '../src/app/BrightSource/ChatContext';

jest.mock('../src/store/BrightSource');
jest.mock('../src/store/Tabs');

test('renders root and can drill down into a folder', async () => {
  // Setup a minimal MobX BrightSource.Directory.allItems
  BrightSource.Directory.allItems = [
    { id: 'f1', name: 'Folder One', docType: 'dir', items: [] }
  ];
  render(
    <ChatAppContext.Provider value={{ BrightSource, Tabs }}>
      <BrightSourceTree />
    </ChatAppContext.Provider>
  );

  // Wait for folder to render
  expect(await screen.findByText('Folder One')).toBeInTheDocument();

  // Click it to drill down
  fireEvent.click(screen.getByText('Folder One'));

  // After drilling down, path breadcrumb should show /home/Folder One
  await waitFor(() => {
    expect(screen.getByText('Folder One')).toBeInTheDocument();
    // ...assert treeData has updated state
  });
});
```

This should get you started on both unit tests (Jest) and component tests (React Testing Library).